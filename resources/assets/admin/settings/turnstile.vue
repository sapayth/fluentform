<template>
    <div>
        <el-row class="setting_header">
            <el-col :md="24">
                <h2>{{ $t('Cloudflare Turnstile Settings') }}</h2>

                <p>
                    {{
                        $t('Fluent Forms integrates with Cloudflare Turnstile, a free service that protects your website from spam and abuse. Please note, these settings are required only if you decide to use the Turnstile field.')
                    }}

                    <a href="https://www.cloudflare.com/en-gb/products/turnstile/" target="_blank">
                        {{ $t('Read more about Cloudflare Turnstile.') }}
                    </a>
                </p>
                <p><b>{{ $t('Please generate API key and API secret using Cloudflare Turnstile') }}</b></p>
            </el-col>
        </el-row>


        <div class="section-body">
            <el-form label-width="205px" label-position="left">

                <!--Site key-->
                <el-form-item>
                    <template slot="label">
                        {{ $t('Site Key') }}
                        <el-tooltip class="item" placement="bottom-start" effect="light">
                            <div slot="content">
                                <h3>{{ $t('Turnstile Site Key') }}</h3>
                                <p>
                                    {{ $t('Enter your Turnstile Site Key, if you do not have ') }}<br />
                                    {{ $t('a key you can register for one at the provided link.') }}<br />
                                    {{ $t('Turnstile is a free service.') }}
                                </p>
                            </div>

                            <i class="el-icon-info el-text-info"></i>
                        </el-tooltip>
                    </template>

                    <el-input v-model="turnstile.siteKey" @change="load"></el-input>
                </el-form-item>

                <!--Secret key-->
                <el-form-item>
                    <template slot="label">
                        {{ $t('Secret Key') }}
                        <el-tooltip class="item" placement="bottom-start" effect="light">
                            <div slot="content">
                                <h3>{{ $t('Turnstile Secret Key') }}</h3>

                                <p>
                                    {{ $t('Enter your Turnstile Secret Key, if you do not have') }}<br>
                                    {{ $t('a key you can register for one at the provided link.') }} <br>
                                    {{ $t('Turnstile is a free service.') }}
                                </p>
                            </div>

                            <i class="el-icon-info el-text-info"></i>
                        </el-tooltip>
                    </template>

                    <el-input type="password" v-model="turnstile.secretKey" @change="load"></el-input>
                </el-form-item>

                <!--Validate Keys-->
                <el-form-item :label="$t('Validate Keys')" v-if="siteKeyChanged">
                    <div
                        class="cf-turnstile"
                        id="turnstile"
                        :data-sitekey="turnstile.siteKey"
                        data-callback="turnstileCallback"
                    ></div>
                </el-form-item>

                <el-form-item>
                    <el-button
                        type="danger"
                        icon="el-icon-delete"
                        size="medium"
                        @click="clearSettings"
                        :loading="clearing"
                    >{{ $t('Clear Settings') }}
                    </el-button>

                    <el-button
                        type="success"
                        icon="el-icon-success"
                        size="medium"
                        @click="save"
                        :disabled="disabled"
                        :loading="saving"
                    >{{ $t('Save Settings') }}
                    </el-button>
                </el-form-item>
            </el-form>

            <div v-if="turnstile_status && !disabled">
                <p>{{ $t('Your Cloudflare Turnstile is valid') }}</p>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "turnstile",
    props: ["app"],
    data() {
        return {
            turnstile: {
                siteKey: "",
                secretKey: "",
            },
            turnstile_status: false,
            siteKeyChanged: false,
            disabled: false,
            saving: false,
            clearing: false,
        };
    },
    methods: {
        load() {
            if (!this.validate()) {
                this.disabled = false;
                this.siteKeyChanged = false;
                return;
            } else {
                this.siteKeyChanged = true;
                this.turnstile_status = false;
            }

            this.$nextTick(() => {
                let id = '#turnstile';
                let $turnstile = jQuery(id);
                let siteKey = this.turnstile.siteKey;
                $turnstile.html('');

                let widgetID = turnstile.render(id, {
                    sitekey: siteKey,
                    callback: (token) => {
                        this.turnstile.token = token;
                    }
                });
            })
        },
        save() {
            if (!this.validate()) {
                return this.$notify.error({
                    title: 'Error!',
                    message: 'Missing required fields.',
                    offset: 30
                });
            }
            this.saving = true;

            FluentFormsGlobal.$post({
                action: 'fluentform-global-settings-store',
                key: 'turnstile',
                turnstile: this.turnstile
            }).then(response => {
                    this.disabled = true;
                    this.turnstile_status = response.data.status;
                    this.$notify.success({
                        title: 'Success!',
                        message: response.data.message,
                        offset: 30
                    });
                })
                .fail(error => {
                    this.turnstile_status = parseInt(error.responseJSON.data.status, 10);
                    let title = this.turnstile_status === 1 ? 'Warning!' : 'Error!';
                    let method = this.turnstile_status === 1 ? 'warning' : 'error';
                    this.$notify[method]({
                        title: title,
                        message: error.responseJSON.data.message,
                        offset: 30
                    });
                }).always(r => {
                this.saving = false;
            });
        },
        clearSettings() {
            this.clearing = true;
            FluentFormsGlobal.$post({
                action: 'fluentform-global-settings-store',
                key: 'turnstile',
                turnstile: 'clear-settings'
            }).then(response => {
                    this.turnstile_status = response.data.status;
                    this.turnstile = {siteKey: '', secretKey: ''};
                    this.$notify.success({
                        title: 'Success!',
                        message: response.data.message,
                        offset: 30
                    });
                })
                .fail(error => {
                    this.turnstile_status = error.responseJSON.data.status;
                    this.$notify.error({
                        title: 'Oops!',
                        message: 'Something went wrong.',
                        offset: 30
                    });
                }).always(r => {
                this.clearing = false;
            });
        },
        validate() {
            return !!(this.turnstile.siteKey && this.turnstile.secretKey);
        },
        getTurnstileSettings() {
            FluentFormsGlobal.$get({
                    action: 'fluentform-global-settings',
                    key: [
                        '_fluentform_turnstile_details',
                        '_fluentform_turnstile_keys_status'
                    ]
                })
                .then(response => {
                    const turnstile = response.data._fluentform_turnstile_details || {siteKey: '', secretKey: ''};
                    this.turnstile = turnstile;
                    this.turnstile_status = response.data._fluentform_turnstile_keys_status;
                });
        }
    },
    mounted() {
        this.getTurnstileSettings();
    },
    created() {
        let turnstileScript = document.createElement('script');

        turnstileScript.setAttribute('src', 'https://challenges.cloudflare.com/turnstile/v0/api.js');

        document.body.appendChild(turnstileScript);
    },
};
</script>