<template>
    <div>
        <template v-if="editItem.columns.length > 1">
            <el-form-item>
                <elLabel slot="label" :label="listItem.label" :helpText="listItem.help_text"></elLabel>
            </el-form-item>

            <template v-for="(column, i) in columns">
                <el-form-item :label="'Column ' + (i + 1)" :key="i">
                    <el-input
                        type="number"
                        :min="minWidth"
                        :max="100"
                        v-model.number="column.width"
                        @change="(value) => set(value, i)"
                    />
                </el-form-item>
            </template>

           <p>{{ listItem.width_limitation_msg }}</p>
        </template>
    </div>
</template>

<script>
    import elLabel from "../../includes/el-label";

    export default {
        name: "containerWidth",
        props: ['listItem', 'editItem', 'value', 'item'],
        components: {
            elLabel
        },

        data() {
            return {
                columns: JSON.parse(JSON.stringify(this.editItem.columns)),
                minWidth: 10
            }
        },

        watch: {
            'editItem.columns': {
                handler (val) {
                    this.columns = JSON.parse(JSON.stringify(val));
                },
                deep: true
            }
        },

        methods: {
            set(value, index) {
                value = this.getNumber(value);
                
                if (value < 10) {
                    return;
                }

                const editingColumn = this.editItem.columns[index];
                const targetColumn = index ? this.editItem.columns[index - 1] : this.editItem.columns[1];
                const availableWdith = targetColumn.width + editingColumn.width;
                const breakingPoint = this.getNumber(availableWdith - this.minWidth);
                
                value = value >= breakingPoint ? breakingPoint : value;

                editingColumn.width = value;
                targetColumn.width = this.getNumber(availableWdith - value);
                this.columns[index].width = value;
            },

            getNumber(value) {
                value = value || 0;

                return +(Math.round(value + "e+2")  + "e-2");
            }
        }
    }
</script>

<style scoped>

</style>
