<script setup lang="ts">
import z from "zod";
import { AutoForm } from "@/components/ui/auto-form";

const formSchema = z.object({
  type: z
    .enum(["my-optional-number", "my-string", "my-enum", "nothing"])
    .default("my-optional-number"),
  foo: z.discriminatedUnion("type", [
    z.object({
      type: z.literal("my-optional-number"),
      data: z.number().default(42).optional(),
    }),
    z.object({
      type: z.literal("my-string"),
      data: z.string().default("some string"),
    }),
    z.object({
      type: z.literal("my-enum"),
      data: z.enum(["foo", "bar", "baz"]).default("foo"),
    }),
    z.object({
      type: z.literal("nothing"),
      data: z.never(),
    }),
  ]),
});
</script>

<template>
  <div class="mx-auto my-3 max-w-xl space-y-6">
    <h1 class="scroll-m-20 text-4xl font-extrabold tracking-tight lg:text-5xl">
      Auto Form with DiscriminatedUnions
    </h1>
    <AutoForm class="space-y-6" :schema="formSchema" />
  </div>
</template>
