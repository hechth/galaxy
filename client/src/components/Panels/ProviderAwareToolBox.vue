<template>
    <ConfigProvider v-slot="{ config }" class="d-flex flex-column">
        <ToolPanelViewProvider
            v-if="config.default_panel_view"
            v-slot="{ currentPanel, currentPanelView }"
            :panel-view="config.default_panel_view">
            <ToolBox
                v-if="currentPanelView"
                :toolbox="currentPanel"
                :panel-views="config.panel_views"
                :current-panel-view="currentPanelView"
                @updatePanelView="updatePanelView">
            </ToolBox>
        </ToolPanelViewProvider>
    </ConfigProvider>
</template>

<script>
import ConfigProvider from "components/providers/ConfigProvider";
import ToolPanelViewProvider from "components/providers/ToolPanelViewProvider";
import { mapActions } from "vuex";

import ToolBox from "./ToolBox";

export default {
    components: {
        ConfigProvider,
        ToolBox,
        ToolPanelViewProvider,
    },
    methods: {
        updatePanelView(panelView) {
            this.setCurrentPanelView(panelView);
        },
        ...mapActions("panels", ["setCurrentPanelView"]),
    },
};
</script>
