<template>
<span>
	<el-popover
            ref="popover"
            placement="top"
            width="160"
            v-model="visible"
    >
		<p>{{ $t('Are you sure to delete this?') }}</p>
		<div style="text-align: right; margin: 0">
			<el-button size="mini" type="text"
                       @click="visible = false"
            >{{ $t('cancel') }}</el-button>

			<el-button type="primary" size="mini"
                       @click="confirmAction"
            >{{ $t('confirm') }}</el-button>
		</div>
	</el-popover>

	<span class="remove-btn" v-popover:popover>
		<slot name="icon">
			<el-button size="mini" type="danger" icon="el-icon-delete" :plain="plain">
				<slot></slot>
			</el-button>
		</slot>
	</span>
</span>
</template>

<script>
    export default {
        name: 'confirmRemove',
        props: {
            plain: {
                type: Boolean,
                default: false
            }
        },
        data() {
            return {
                visible: false
            };
        },
        methods: {
            confirmAction() {
                this.visible = false;
                this.$emit('on-confirm');
            }
        }
    };
</script>
