<script setup lang="ts">
import { computed } from "vue";

import { bytesToString } from "@/utils/utils";

import type { DataValuePoint } from "./Charts";

interface RecoverableItemSizeTooltipProps {
    data: DataValuePoint;
    isRecoverable: boolean;
}

const props = defineProps<RecoverableItemSizeTooltipProps>();

const label = computed(() => props.data?.label ?? "No data");
const prettySize = computed(() => bytesToString(props.data?.value ?? 0));
</script>
<template>
    <div>
        <div class="h-md mx-2">{{ label }}</div>
        <b class="h-md m-2">{{ prettySize }}</b>
        <div v-if="isRecoverable" class="text-muted mx-2">Recoverable storage space</div>
    </div>
</template>
