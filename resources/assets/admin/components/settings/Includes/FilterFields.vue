<template>
    <div>
        <el-checkbox v-model="conditionals.status" v-if="!disabled">
            {{ labels.status_label }}
        </el-checkbox>

        <el-checkbox v-else disabled @click.native="comingSoon = true">
            {{ labels.status_label }}
        </el-checkbox>

        <div v-if="conditionals.status">
            {{ labels.notification_if_start }}

            <select class="ff_inline_small" v-model="conditionals.type">
                <option v-for="(label, value) in {all: 'All', any: 'Any'}" :key="value" :value="value">
                    {{ label }}
                </option>
            </select>

            {{ labels.notification_if_end }}

            <el-row v-for="(logic, key) in items" :key="key"
                    style="margin-top: 15px;" :gutter="10"
            >
                <el-col :md="8">
                    <el-select v-model="items[key].field" style="width: 100%" @change="items[key].value = ''">
                        <el-option v-for="(field, key) in fields" :key="key"
                                   :label="field.admin_label" :value="key"
                        ></el-option>
                    </el-select>
                </el-col>

                <el-col :md="5">
                    <el-select v-model="items[key].operator">
                        <el-option-group :label="$t('General Operators')">
                            <el-option value="=" :label="$t('equal')"></el-option>
                            <el-option value="!=" :label="$t('not equal')"></el-option>
                            <template v-if="fields[logic.field] && !Object.keys(fields[logic.field].options).length">
                                <el-option value=">" :label="$t('greater than')"></el-option>
                                <el-option value="<" :label="$t('less than')"></el-option>
                                <el-option value=">=" :label="$t('greater than or equal')"></el-option>
                                <el-option value="<=" :label="('less than or equal')"></el-option>
                                <el-option value="contains" :label="$t('contains')"></el-option>
                                <el-option value="doNotContains" :label="$t('do not contains')"></el-option>
                                <el-option value="startsWith" :label="$t('starts with')"></el-option>
                                <el-option value="endsWith" :label="$t('ends with')"></el-option>
                            </template>
                        </el-option-group>
                        <el-option-group :label="$t('Advanced Operators')">
                            <el-option value="length_equal" :label="$t('Equal to Data Length')"></el-option>
                            <el-option value="length_less_than" :label="$t('Less than to Data length')"></el-option>
                            <el-option value="length_greater_than" :label="$t('Greater than to Data Length')"></el-option>
                            <el-option value="test_regex" :label="$t('Regex Match')"></el-option>
                        </el-option-group>
                    </el-select>
                </el-col>

                <el-col :md="8">
                    <template v-if="items[key].operator == 'length_equal' || items[key].operator == 'length_less_than' || items[key].operator == 'length_greater_than'">
                        <el-input type="number" step="1" :placeholder="('Enter length in number')" v-model="items[key].value" />
                    </template>
                    <template v-else>
                        <el-select v-if="fields[logic.field] && Object.keys(fields[logic.field].options).length"
                                   v-model="items[key].value" style="width: 100%">
                            <el-option v-for="(label, value) in fields[logic.field].options" :key="value"
                                       :label="label" :value="value"
                            ></el-option>
                        </el-select>
                        <el-input v-else :placeholder="$t('Enter a value')" v-model="items[key].value"></el-input>
                    </template>
                </el-col>

                <el-col :md="3" class="action-btns ">
                    <i class="el-icon-plus" @click="add(key)"></i>

                    <i class="el-icon-minus" @click="remove(key)"
                       v-if="items.length > 1"
                    ></i>
                </el-col>
            </el-row>
        </div>

        <coming-soon
            v-if="disabled"
            :visibility.sync="comingSoon"/>

    </div>
</template>

<script>
import ComingSoon from '../../modals/ItemDisabled';

export default {
    name: 'FilterFields',
    components: {
        ComingSoon
    },
    props: {
        conditionals: {
            type: Object,
            required: true,
            default: {}
        },
        fields: {
            type: Object,
            required: true,
            default: {}
        },
        disabled: {
            type: Boolean,
            required: true,
            default: false,
        },
        labels: {
            default: () => ({
                status_label: 'Enable conditional logic',
                notification_if_start: 'Send this notification if',
                notification_if_end: 'of the following match:'
            })
        }
    },
    data() {
        return {
            defaultRules: {
                field: null,
                operator: '=',
                value: null
            },
            comingSoon: false,
        }
    },
    computed: {
        items() {
            return this.conditionals.conditions;
        }
    },
    methods: {
        add(index) {
            this.items.splice(index + 1, 0, {...this.defaultRules});
        },
        remove(index) {
            this.items.splice(index, 1);
        }
    },
    mounted() {
        if (!this.conditionals.conditions.length) {
            this.conditionals.conditions.push({...this.defaultRules});
        }
    }
};
</script>
