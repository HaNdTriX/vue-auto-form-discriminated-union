<script setup lang="ts" generic="T extends ZodDiscriminatedUnion<string, any>">
import type { ZodAny, ZodDiscriminatedUnion } from "zod";
import { computed, provide } from "vue";
import { FieldContextKey, useField } from "vee-validate";
import AutoFormField from "./AutoFormField.vue";
import type { ConfigItem, Shape } from "./interface";
import { getBaseSchema, getBaseType, getDefaultValueInZodStack } from "./utils";
import { useFieldValue } from "vee-validate";

const props = defineProps<{
  fieldName: string;
  required?: boolean;
  config?: ConfigItem;
  schema?: T;
  disabled?: boolean;
}>();

const discriminatorFieldValue = useFieldValue<string>(
  () => props.schema!.discriminator
);

const shape = computed(() => {
  if (!props.schema) return;
  if (!props.schema.optionsMap) return;

  const descriminatorOption = Array.from(props.schema.optionsMap).find(
    ([option, _schema]) => option === discriminatorFieldValue.value
  );

  if (!descriminatorOption) {
    return;
  }

  const discriminatorSchema = descriminatorOption[1].shape.data as ZodAny;

  return {
    type: getBaseType(discriminatorSchema),
    default: getDefaultValueInZodStack(discriminatorSchema),
    required: !["ZodOptional", "ZodNullable"].includes(
      discriminatorSchema._def.typeName
    ),
    schema: getBaseSchema(discriminatorSchema) || undefined,
  } satisfies Shape;
});

const fieldContext = useField(props.fieldName);
// @ts-expect-error ignore missing `id`
provide(FieldContextKey, fieldContext);
</script>

<template>
  <AutoFormField
    v-if="shape"
    :key="shape.default"
    :config="config"
    :field-name="fieldName"
    :shape="shape"
  />
</template>
