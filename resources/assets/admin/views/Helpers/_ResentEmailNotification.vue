<template>
    <div class="ff_email_resend_inline">
        <el-button v-if="element_type == 'button'" @click="dialogVisible=true" type="info" size="small">{{ $t(btn_text) }}</el-button>
        <el-dialog
            :title="$t('Choose Email Notification')"
            top="42px"
            @before-close="resetData()"
            :append-to-body="true"
            :visible.sync="dialogVisible"
            width="60%">
            <template v-if="has_pro">
                <el-form label-width="120px" ref="form" :model="form" label-position="left">
                    <el-form-item :label="$t('Notification')">
                        <el-select size="small" :placeholder="$t('Select Notification')"
                                   v-model="form.selected_notification_id">
                            <el-option v-for="notification in notifications" :value="notification.id"
                                       :label="notification.name" :key="notification.id"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item :label="$t('Send To')">
                        <el-radio-group size="small" v-model="form.send_to_type">
                            <el-radio-button label="default">{{ $t('Default Recipient') }}</el-radio-button>
                            <el-radio-button label="custom">{{ $t('Custom Recipient') }}</el-radio-button>
                        </el-radio-group>
                    </el-form-item>
                    <template v-if="form.send_to_type == 'custom'">
                        <el-form-item :label="$t('Recipient')">
                            <el-input v-model="form.send_to_custom_email" size="small"
                                      :placeholder="$t('Please Type Recipient Email Address')"/>
                        </el-form-item>
                    </template>
                </el-form>
                <div v-if="error_message" v-html="error_message" class="ff-error"></div>
                <div v-if="success_message" v-html="success_message" class="ff-success"></div>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="dialogVisible = false">{{ $t('Cancel') }}</el-button>
                    <el-button v-loading="sending" :disabled="!isActive" type="primary" @click="send()">{{ $t('Resend this notification') }}</el-button>
                </span>
            </template>
            <div style="text-align: center" v-else>
                <h3>{{ $t('This feature is available on pro version of fluent forms.') }}</h3>
                <a target="_blank" rel="noopener" :href="upgrade_url" class="el-button el-button--danger">
                    {{ $t('Buy Pro Now') }}
                </a>
            </div>
        </el-dialog>
    </div>
</template>
<script type="text/babel">
    export default {
        name: 'resentEmailNotification',
        props: {
            entry_id: {
                default() {
                    return '';
                }
            },
            form_id: {
                required: true
            },
            entry_ids: {
                default() {
                    return []
                }
            },
            element_type: {
                default() {
                    return 'button'
                }
            },
            btn_text: {
                default() {
                    return 'Resend Email Notification'
                }
            }
        },
        data() {
            return {
                has_pro: !!window.fluent_form_entries_vars.has_pro,
                dialogVisible: false,
                sending: false,
                error_message: '',
                success_message: '',
                form: {
                    selected_notification_id: '',
                    send_to_type: 'default',
                    send_to_custom_email: ''
                },
                notifications: window.fluent_form_entries_vars.email_notifications,
                upgrade_url: window.fluent_form_entries_vars.upgrade_url
            }
        },
        computed: {
            isActive() {
                if (this.form.send_to_type == 'custom') {
                    return this.form.selected_notification_id && this.form.send_to_custom_email;
                }
                return this.form.selected_notification_id;
            }
        },
        methods: {
            send() {
                if (this.sending) {
                    return;
                }
                this.sending = true;
                this.error_message = '';
                this.success_message = '';
                let data = {
                    action: 'ffpro-resent-email-notification',
                    notification_id: this.form.selected_notification_id,
                    form_id: this.form_id,
                    entry_id: this.entry_id,
                    entry_ids: this.entry_ids,
                    send_to_type: this.form.send_to_type,
                    send_to_custom_email: this.form.send_to_custom_email,
                    ff_sumulate: 'fluentform_submit'
                };
                FluentFormsGlobal.$post(data)
                    .then(response => {
                        this.$notify.success(response.data.message);
                        this.success_message = response.data.message;
                        this.form = {
                            selected_notification_id: '',
                            send_to_type: 'default',
                            send_to_custom_email: ''
                        };
                        this.$emit('reloadLogs', 1);
                    })
                    .fail(error => {
                        if (!error.responseJSON && !error.responseText || error.responseText == '0') {
                            alert(this.$t('Looks like you are using older version of fluent forms pro. Please update to latest version'));
                            return;
                        }
                        this.error_message = error.responseJSON.data.message;
                    })
                    .always(() => {
                        this.sending = false;
                    });
            },
            resetData() {
                this.error_message = '';
                this.success_message = '';
                this.form = {
                    selected_notification_id: '',
                    send_to_type: 'default',
                    send_to_custom_email: ''
                }
            }
        }
    }
</script>
