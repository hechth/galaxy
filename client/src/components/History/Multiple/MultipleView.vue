<script setup lang="ts">
import { library } from "@fortawesome/fontawesome-svg-core";
import { faTimes } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { storeToRefs } from "pinia";
import { computed, onMounted, type Ref, ref, watchEffect, type WatchStopHandle } from "vue";

import { useHistoryStore } from "@/stores/historyStore";
import { useUserStore } from "@/stores/userStore";
import localize from "@/utils/localization";

import MultipleViewList from "./MultipleViewList.vue";
import SelectorModal from "@/components/History/Modals/SelectorModal.vue";
import LoadingSpan from "@/components/LoadingSpan.vue";

const filter = ref("");
const showSelectModal = ref(false);

library.add(faTimes);

const { currentUser } = storeToRefs(useUserStore());
const { histories, currentHistory } = storeToRefs(useHistoryStore());

const historyStore = useHistoryStore();
const selectedHistories: Ref<{ id: string }[]> = computed(() => historyStore.pinnedHistories);

const noHistoriesInView = computed(() => !selectedHistories.value.length && histories.value.length > 0);

const loadingPinnedHistories = ref(true);
const stopStoreWatcher: Ref<WatchStopHandle | null> = ref(null);
onMounted(async () => {
    loadingPinnedHistories.value = true;
    stopStoreWatcher.value = watchEffect(() => {
        if (noHistoriesInView.value == true) {
            historyStore.pinHistory(histories.value[0]!.id);
            if (stopStoreWatcher.value) {
                stopStoreWatcher.value();
            }
        }
    });
    loadingPinnedHistories.value = false;
});

/**
 * From the incoming list, pin histories that are not already pinned and
 * unpin histories that are not in the incoming list.
 * This is a mechanism to allow users to select or deselect histories in modal.
 * @param histories the incoming list of histories to pin
 */
function addHistoriesToList(histories: { id: string }[]) {
    // Unpin histories that are already pinned but not in the incoming list
    const historiesToUnpin = selectedHistories.value.filter(
        (pinnedHistory) => !histories.some((history) => history.id === pinnedHistory.id)
    );
    historiesToUnpin.forEach((history) => {
        historyStore.unpinHistory(history.id);
    });
    // Pin histories that aren't already pinned and are in incoming list
    histories.forEach((history) => {
        const historyExists = selectedHistories.value.some((h) => h.id === history.id);
        if (!historyExists) {
            historyStore.pinHistory(history.id);
            historyStore.loadHistoryById(history.id);
        }
    });
}

function updateFilter(newFilter: string) {
    filter.value = newFilter;
}
</script>

<template>
    <div v-if="currentUser">
        <b-alert v-if="loadingPinnedHistories" class="m-2" variant="info" show>
            <LoadingSpan message="Loading Histories" />
        </b-alert>
        <div v-else-if="histories.length" class="multi-history-panel d-flex flex-column h-100">
            <b-input-group class="w-100">
                <b-form-input
                    v-model="filter"
                    size="sm"
                    debounce="500"
                    :class="filter && 'font-weight-bold'"
                    :placeholder="localize('search datasets in selected histories')"
                    data-description="filter text input"
                    @keyup.esc="updateFilter('')" />
                <b-input-group-append>
                    <b-button size="sm" data-description="show deleted filter toggle" @click="updateFilter('')">
                        <FontAwesomeIcon icon="fa-times" />
                    </b-button>
                </b-input-group-append>
            </b-input-group>
            <MultipleViewList
                :histories="histories"
                :filter="filter"
                :current-history="currentHistory"
                :selected-histories="selectedHistories"
                :show-modal.sync="showSelectModal" />
        </div>
        <b-alert v-else class="m-2" variant="danger" show>
            <span v-localize class="font-weight-bold">No History found.</span>
        </b-alert>
        <SelectorModal
            v-show="showSelectModal"
            :multiple="true"
            :histories="histories"
            :additional-options="['center', 'set-current']"
            :show-modal.sync="showSelectModal"
            title="Select/Deselect histories"
            @selectHistories="addHistoriesToList" />
    </div>
</template>
