<template>
    <div>
        <vddl-draggable class="panel__body--item js-editor-item"
                        :class="{ 'selected': editItem.uniqElKey == item.uniqElKey }"
                        :draggable="item"
                        :index="index"
                        effect-allowed="move"
                        type="existingElement"
                        :dragstart="handleDragstart"
                        :dragend="handleDragend"
                        :moved="handleMoved"
                        :wrapper="wrapper">

            <div @click="editSelected(index, item)" class="item-actions-wrapper"
                 :class="item.element == 'container' ? 'hover-action-top-right' : 'hover-action-middle'">
                <div class="item-actions">
                    <i class="icon icon-arrows"></i>
                    <i @click="editSelected(index, item)" class="icon icon-pencil"></i>
                    <i @click="duplicateSelected(index, item)" class="icon icon-clone"></i>
                    <i @click="askRemoveConfirm(index)" class="icon icon-trash-o"></i>
                    <i
                        v-show="item.element == 'container' && item.columns.length > 1"
                        @click="resetContainer()"
                        class="icon ff-edit-repeat"
                    />
                </div>
            </div>

            <i @click.stop="editorInserterPopup(index, wrapper)" class="popup-search-element">+</i>

            <div v-if="item.element == 'container'" class="item-container">
                <div class="ff_condition_icon" v-html="maybeConditionIcon(item.settings)"></div>
                <vddl-nodrag style="width: 100%">
                    <splitpanes
                                class="default-theme"
                                @resized="resize($event)"
                        >
                            <pane
                                v-for="(containerRow, i) in item.columns"
                                :key="i"
                                :size="item.columns[i].width <= 10 ? 10 : item.columns[i].width"
                                min-size="10"
                            >
                                <vddl-list class="panel__body"
                                        :list="containerRow.fields"
                                        :drop="handleDrop"
                                        :horizontal="false">

                                    <div v-show="!containerRow.fields.length" style="padding-top: 13px;"
                                        class="empty-dropzone-placeholder">
                                        <i @click.stop="editorInserterPopup(0, containerRow.fields)"
                                        class="popup-search-element">+</i>
                                    </div>
                                    <list v-for="(field, list_index) in containerRow.fields"
                                        :key="field.uniqElKey"
                                        :item="field"
                                        :index="list_index"
                                        :handleEdit="handleEdit"
                                        :allElements="allElements"
                                        :editItem="editItem"
                                        :wrapper="containerRow.fields">
                                    </list>
                                </vddl-list>
                            </pane>
                        </splitpanes>  
                </vddl-nodrag>

            </div>

            <template v-if="item.element != 'container' && hasRegistered(item)">
                <div class="ff_condition_icon" v-html="maybeConditionIcon(item.settings)"></div>
                <component :is="guessElTemplate(item)" :item="item"></component>
                <p style="font-style: italic;" v-if="item.settings.help_message" class="help-text"
                   v-html="item.settings.help_message"></p>
            </template>
        </vddl-draggable>

        <ff_removeElConfirm
                :editItem="editItem"
                :visibility.sync="showRemoveElConfirm"
                @on-confirm="onRemoveElConfirm"/>
    </div>
</template>

<script type="text/babel">
    import NestedHandler from "./NestedHandler.js";
    export default {
        name: 'list',
        props: NestedHandler.props,
        components: NestedHandler.components,
        data() {
            return {
                showRemoveElConfirm: false,
                removeElIndex: null,
            }
        },
        methods: NestedHandler.methods
    };
</script>
