<template>
    <b-card id="lint-panel" header-tag="header" body-class="p-0" class="right-content">
        <template v-slot:header>
            <div class="mb-1 font-weight-bold">
                <FontAwesomeIcon icon="magic" class="mr-1" />
                Best Practices Review
            </div>
            <div v-if="showRefactor">
                <a href="#" @click="onRefactor"> Try to automatically fix issues. </a>
            </div>
        </template>
        <b-card-body>
            <LintSection
                :okay="checkAnnotation"
                success-message="This workflow is annotated. Ideally, this helps the executors of the workflow
                    understand the purpose and usage of the workflow."
                warning-message="This workflow is not annotated. Providing an annotation helps workflow executors
                    understand the purpose and usage of the workflow."
                attribute-link="Annotate your Workflow."
                @onClick="onAttributes" />
            <LintSection
                :okay="checkCreator"
                success-message="This workflow defines creator information."
                warning-message="This workflow does not specify creator(s). This is important metadata for workflows
                    that will be published and/or shared to help workflow executors know how to cite the
                    workflow authors."
                attribute-link="Provide Creator Details."
                @onClick="onAttributes" />
            <LintSection
                :okay="checkLicense"
                success-message="This workflow defines a license."
                warning-message="This workflow does not specify a license. This is important metadata for workflows
                    that will be published and/or shared to help workflow executors understand how it
                    may be used."
                attribute-link="Specify a License."
                @onClick="onAttributes" />
            <LintSection
                success-message="Workflow parameters are using formal input parameters."
                warning-message="This workflow uses legacy workflow parameters. They should be replaced with
                formal workflow inputs. Formal input parameters make tracking workflow provenance, usage within subworkflows,
                and executing the workflow via the API more robust:"
                :warning-items="warningUntypedParameters"
                @onMouseOver="onHighlight"
                @onMouseLeave="onUnhighlight"
                @onClick="onFixUntypedParameter" />
            <LintSection
                success-message="All non-optional inputs to workflow steps are connected to formal input parameters."
                warning-message="Some non-optional inputs are not connected to formal workflow inputs. Formal input parameters
                make tracking workflow provenance, usage within subworkflows, and executing the workflow via the API more robust:"
                :warning-items="warningDisconnectedInputs"
                @onMouseOver="onHighlight"
                @onMouseLeave="onUnhighlight"
                @onClick="onFixDisconnectedInput" />
            <LintSection
                success-message="All workflow inputs have labels and annotations."
                warning-message="Some workflow inputs are missing labels and/or annotations:"
                :warning-items="warningMissingMetadata"
                @onMouseOver="onHighlight"
                @onMouseLeave="onUnhighlight"
                @onClick="onScrollTo" />
            <LintSection
                success-message="This workflow has outputs and they all have valid labels."
                warning-message="The following workflow outputs have no labels, they should be assigned a useful label or
                    unchecked in the workflow editor to mark them as no longer being a workflow output:"
                :warning-items="warningUnlabeledOutputs"
                @onMouseOver="onHighlight"
                @onMouseLeave="onUnhighlight"
                @onClick="onFixUnlabeledOutputs" />
            <div v-if="!hasActiveOutputs">
                <FontAwesomeIcon icon="exclamation-triangle" class="text-warning" />
                <span>This workflow has no labeled outputs, please select and label at least one output.</span>
            </div>
        </b-card-body>
    </b-card>
</template>

<script>
import { library } from "@fortawesome/fontawesome-svg-core";
import { faExclamationTriangle, faMagic } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import BootstrapVue from "bootstrap-vue";
import LintSection from "components/Workflow/Editor/LintSection";
import { UntypedParameters } from "components/Workflow/Editor/modules/parameters";
import { storeToRefs } from "pinia";
import Vue from "vue";

import { DatatypesMapperModel } from "@/components/Datatypes/model";
import { useWorkflowStepStore } from "@/stores/workflowStepStore";

import {
    fixAllIssues,
    fixDisconnectedInput,
    fixUnlabeledOutputs,
    fixUntypedParameter,
    getDisconnectedInputs,
    getMissingMetadata,
    getUnlabeledOutputs,
    getUntypedParameters,
} from "./modules/linting";

Vue.use(BootstrapVue);

library.add(faExclamationTriangle);
library.add(faMagic);

export default {
    components: {
        FontAwesomeIcon,
        LintSection,
    },
    props: {
        untypedParameters: {
            type: UntypedParameters,
            required: true,
        },
        steps: {
            type: Object,
            required: true,
        },
        annotation: {
            type: String,
            default: null,
        },
        license: {
            type: String,
            default: null,
        },
        creator: {
            type: Array,
            default: null,
        },
        datatypesMapper: {
            type: DatatypesMapperModel,
            required: true,
        },
    },
    setup() {
        const { hasActiveOutputs } = storeToRefs(useWorkflowStepStore());
        return { hasActiveOutputs };
    },
    computed: {
        showRefactor() {
            // we could be even more precise here and check the inputs and such, because
            // some of these extractions may not be possible.
            return !this.checkUntypedParameters || !this.checkDisconnectedInputs || !this.checkUnlabeledOutputs;
        },
        checkAnnotation() {
            return !!this.annotation;
        },
        checkLicense() {
            return !!this.license;
        },
        checkCreator() {
            if (this.creator instanceof Array) {
                return this.creator.length > 0;
            } else {
                return !!this.creator;
            }
        },
        checkUntypedParameters() {
            return this.warningUntypedParameters.length == 0;
        },
        checkDisconnectedInputs() {
            return this.warningDisconnectedInputs.length == 0;
        },
        checkUnlabeledOutputs() {
            return this.warningUnlabeledOutputs.length == 0;
        },
        warningUntypedParameters() {
            return getUntypedParameters(this.untypedParameters);
        },
        warningDisconnectedInputs() {
            return getDisconnectedInputs(this.steps, this.datatypesMapper);
        },
        warningMissingMetadata() {
            return getMissingMetadata(this.steps);
        },
        warningUnlabeledOutputs() {
            return getUnlabeledOutputs(this.steps);
        },
    },
    methods: {
        onAttributes() {
            this.$emit("onAttributes");
        },
        onFixUntypedParameter(item) {
            if (
                confirm(
                    "This issue can be fixed automatically by creating an explicit parameter input step. Do you want to proceed?"
                )
            ) {
                this.$emit("onRefactor", [fixUntypedParameter(item)]);
            } else {
                this.$emit("onScrollTo", item.stepId);
            }
        },
        onFixDisconnectedInput(item) {
            if (
                confirm(
                    "This issue can be fixed automatically by creating an explicit data input step. Do you want to proceed?"
                )
            ) {
                this.$emit("onRefactor", [fixDisconnectedInput(item)]);
            } else {
                this.$emit("onScrollTo", item.stepId);
            }
        },
        onFixUnlabeledOutputs(item) {
            if (
                confirm(
                    "This issue can be fixed automatically by removing all unlabeled workflow output. Do you want to proceed?"
                )
            ) {
                this.$emit("onRefactor", [fixUnlabeledOutputs()]);
            } else {
                this.$emit("onScrollTo", item.stepId);
            }
        },
        onScrollTo(item) {
            this.$emit("onScrollTo", item.stepId);
        },
        onHighlight(item) {
            this.$emit("onHighlight", item.stepId);
        },
        onUnhighlight(item) {
            this.$emit("onUnhighlight", item.stepId);
        },
        onRefactor() {
            const actions = fixAllIssues(this.steps, this.untypedParameters);
            this.$emit("onRefactor", actions);
        },
    },
};
</script>
