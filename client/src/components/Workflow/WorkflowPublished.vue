<template>
    <Published :item="workflow">
        <template v-slot>
            <WorkflowDisplay :workflow-id="id" :expanded="true" />
        </template>
    </Published>
</template>

<script>
import Published from "components/Common/Published";
import WorkflowDisplay from "components/Markdown/Elements/Workflow/WorkflowDisplay";
import { urlData } from "utils/url";

export default {
    components: {
        Published,
        WorkflowDisplay,
    },
    props: {
        id: {
            type: String,
            required: true,
        },
    },
    data() {
        return {
            workflow: {},
        };
    },
    created() {
        const url = `/api/workflows/${this.id}`;
        urlData({ url }).then((data) => {
            this.workflow = data;
        });
    },
};
</script>
