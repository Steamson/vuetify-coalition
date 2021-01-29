<template>
    <v-form class="file_loader">
        <div class="block_form d-flex flex-column flex-sm-row">
            <div class="column selector mb-5 mb-sm-0 mr-sm-2 flex-shrink-0"
                :class="$vuetify.breakpoint.xs ? 'full' : ''"
            >
                <div class="column_title mb-4">Выбрать источник</div>

                <v-select
                    append-icon="keyboard_arrow_down"
                    :prepend-inner-icon='select.selected'
                    attach=".selector__input"
                    @change="clearForms()"
                    class="selector__input"
                    v-model="select.selected"
                    :items="select.options"
                    absolute="true"
                    hide-details
                    outlined
                    dense
                >
                    <template v-slot:item="slotProps">
                        <v-icon :class="['mr-2']" size="16">{{slotProps.item.value}}</v-icon>
                        {{slotProps.item.text}}
                    </template>
                </v-select>
            </div>

            <div class="column flex-grow-1">
                <v-item-group>
                    <div v-if="select.selected === 'attach_file'">
                        <div class="column_title mb-4">Загрузите файл</div>

                        <div class="select_file">
                            <div
                                class="select_file__card d-flex align-center justify-space-between rounded pa-5"
                                v-cloak @drop.prevent="droppedFiles" @dragover.prevent
                                :class="files.files_validate"
                                :loading="files.file_choising"
                                @click="onButtonClick"
                                color="primary"
                                depressed
                            >
                                <div class="select_file__card__info mr-3">
                                    Файлы {{fileFormats}}
                                </div>
                                
                                <div class="select_file__card__btn rounded py-2 px-5 flex-shrink-0">
                                    <v-icon :class="['mr-2']" size="12" color="deepBlue">attach_file</v-icon>
                                    <span>Загрузить файлы</span>
                                </div>
                            </div>

                            <input
                                ref="uploader"
                                class="d-none"
                                type="file"
                                @change="onFileChanged"
                            ><!-- accept="image/*" -->

                            <ul v-if="files.selected_files.length" class="select_file__list pl-0">
                                <li
                                    class="added_file d-flex align-center"
                                    v-for="(file, key) in files.selected_files"
                                    :key="key"
                                >
                                    <div class="added_file__icon">
                                        <v-icon>insert_drive_file</v-icon>
                                        <span>{{removeName(file.name)}}</span>
                                    </div>

                                    <span class="added_file__text flex-grow-1 px-5">{{removeFormat(file.name)}}</span>

                                    <v-icon
                                        @click="removeDocument(key)"
                                        class="added_file__close"
                                        size="10"
                                    >clear</v-icon>
                                </li>
                            </ul>
                        </div>
                    </div>

                    <div v-else-if="select.selected === 'link'">
                        <div class="column_title mb-4">Вставить ссылку</div>

                        <div>
                            <v-text-field
                                :append-icon="linkStatus"
                                :class="linkStatus"
                                v-model="link.link_text"
                                class="select_link"
                                hide-details
                                outlined
                                dense
                            ></v-text-field>
                        </div>
                    </div>

                    <div v-else>
                        <div class="column_title mb-4">Вставить code &lt;/&gt;</div>

                        <div>
                            <v-textarea
                                :hide-details="iFrameStatus === 'remove_circle' ? false : true"
                                :rules="[rules.iframe]"
                                :append-icon="iFrameStatus"
                                :class="iFrameStatus"
                                v-model="iframe.iframe_text"
                                class="select_iframe"
                                row-height="14px"
                                height="304px"
                                no-resize
                                outlined
                            ></v-textarea>
                        </div>
                    </div>
                </v-item-group>
            </div>
        </div>

        <div class="block_btn">
            <v-btn
                :class="$vuetify.breakpoint.xs ? 'full' : ''"
                color="deepBlue white--text"
                :disabled="!totalValidate"
                :loading="sending"
                @click="postHandler"
            >Сохранить</v-btn>
        </div>
    </v-form>
</template>

<script>
    import axios from 'axios';

    export default {
        data() {
            return {
                select: {
                    selected: 'attach_file',
                    options: [
                        { text: 'Файл', value: 'attach_file' },
                        { text: 'iFrame', value: 'airplay' },
                        { text: 'Ссылка', value: 'link' },
                    ],
                },
                link: {
                    link_text: '',
                },
                files: {
                    files_validate: '',
                    selected_files: [],
                    file_choising: false,
                    files_supported: ['doc', 'docx', 'pdf', 'word', 'png', 'jpg'],
                },
                iframe: {
                    iframe_text: '',
                },
                sending: false,
                rules: {
                    iframe: () => {
                        return (this.iFrameStatus != 'remove_circle') || 'Сode is not valid';
                    },
                },
            }
        },
        
        computed: {
            linkStatus() {
                let status = '';

                if (this.link.link_text.length) {
                    try {
                        new URL(this.link.link_text);
                        status = 'check_circle';
                    } catch (e) {
                        status = 'remove_circle';
                    }
                }

                return status;
            },

            fileFormats() {
                let string = '';

                for (let i = 0; i < this.files.files_supported.length; i++) {
                    let delimeter = (i === 0) ? '' : ' , ';
                    string += delimeter + '.' + this.files.files_supported[i];
                }

                return string;
            },

            iFrameStatus() {
                let error = null;

                if (this.iframe.iframe_text.length) {
                    if (this.iframe.iframe_text.includes('iframe') && this.iframe.iframe_text.includes('/iframe')) {
                        let test_element = document.createElement('div');
                        test_element.innerHTML = this.iframe.iframe_text;
                        let element = test_element.childNodes[0];

                        if (element.attributes['src'] && element.attributes['src']['value'].length > 0) {
                            try {
                                new URL(element.attributes['src']['value']);
                                error = false;
                            } catch(e) {
                                error = true;
                            }
                            
                            test_element.remove();
                        }
                    } else {
                        error = true;
                    }
                }

                if (error === false) {
                    return 'check_circle';
                } else if (error === true) {
                    return 'remove_circle';
                } else {
                    return '';
                }
            },

            totalValidate() {
                let valid = false;

                switch (this.select.selected) {
                    case 'attach_file':
                        if (this.files.selected_files.length > 0) { valid = true; } break;
                    case 'airplay':
                        if (this.iFrameStatus === 'check_circle') { valid = true; } break;
                    case 'link':
                        if (this.linkStatus == 'check_circle') { valid = true; } break;
                }

                return valid;
            },
        },

        methods: {
            clearForms() {
                this.link.link_text = '';

                this.files.files_validate = '';
                this.files.selected_files = [];

                this.iframe.iframe_text = ''
            },

            onButtonClick() {
                this.files.file_choising = true
                window.addEventListener('focus', () => {
                    this.files.file_choising = false
                }, { once: true })

                this.$refs.uploader.click()
            },

            droppedFiles(e) {
                e.target.files = e.dataTransfer.files;
                this.onFileChanged(e);
            },

            onFileChanged(e) {
                if (e.target.files[0]) {
                    if (this.files.files_supported.indexOf(this.removeName(e.target.files[0].name)) >= 0) {
                        this.files.selected_files.push(e.target.files[0]);
                        this.files.files_validate = 'check_circle';
                    } else {
                        this.files.files_validate = 'remove_circle';
                    }
                } else {
                    this.files.files_validate = '';
                }
            },

            removeDocument(index){
                this.files.selected_files.splice(index, 1);
            },

            removeName(name) {
                return name.split('.').splice(-1, 1).join('.');
            },

            removeFormat(name) {
                return name.split('.').slice(0, -1).join('.');
            },

            postHandler() {
                this.sending = true;
                
                switch (this.select.selected) {
                    case 'attach_file': {
                        let form_data = new FormData();
                        this.files.selected_files.forEach(file => form_data.append('file', file));

                        axios.post('https://httpbin.org/anything', form_data, {
                            headers: { 'Content-Type': 'multipart/form-data' }
                        }).then(() => {
                            this.clearForms();
                            this.sending = false;
                            console.log('SUCCESS File');
                        }).catch(() => {
                            this.sending = false;
                            console.log('ERROR File');
                        });
                        break;
                    }
                    case 'airplay': {
                        let test_element = document.createElement('div');
                        test_element.innerHTML = this.iframe.iframe_text;
                        let element = test_element.childNodes[0];
                        let src = element.attributes['src']['value'];
                        test_element.remove();

                        axios.post('https://httpbin.org/anything', {
                            jsonData: JSON.stringify({
                                type: 'iframe',
                                url: src,
                            })
                        }).then(() => {
                            this.clearForms();
                            this.sending = false;
                            console.log('SUCCESS iFrame');
                        }).catch(() => {
                            this.sending = false;
                            console.log('ERROR iFrame');
                        });
                        break;
                    }
                    case 'link': {
                        axios.post('https://httpbin.org/anything', {
                            jsonData: JSON.stringify({
                                type: 'link',
                                url: this.link.link_text,
                            })
                        }).then(() => {
                            this.clearForms();
                            this.sending = false;
                            console.log('SUCCESS Link');
                        }).catch(() => {
                            this.sending = false;
                            console.log('ERROR Link');
                        });
                        break;
                    }
                }
            }
        }
    }
</script>

<style lang="scss" scoped>
    @import '../assets/styles/vars.scss';

    .file_loader {
        font-size: .7rem;

        .block {
            &_form {
                .selector {
                    width: 34.4%;

                    &.full {
                        width: 100%;
                    }
                    
                    &__input {
                        line-height: .7rem;
                        position: relative;
                        font-size: .6rem;

                        ::v-deep .v-input__control .v-input__slot {
                            padding: 0 .8rem;

                            .v-input__prepend-inner {
                                margin-right: .4rem;
                                margin-top: .45rem;
                                width: $size-icon;
                                padding: 0;

                                .v-input__icon {
                                    min-width: $size-icon;
                                    width: $size-icon;

                                    .v-icon {
                                        font-size: $size-icon;
                                    }
                                }
                            }

                            fieldset {
                                border-color: $color-border;
                            }

                            .v-select__slot .v-input__append-inner .v-input__icon i {
                                padding-top: 0.1rem;
                                color: $color-dark;
                            }
                        }
                    }
                }

                .column {
                    &_title {
                        line-height: 0.8295rem;
                    }

                    .select {
                        &_file {
                            &.v-text-field {
                                padding-top: 0;
                                margin-top: 0;
                            }

                            &__card {
                                transition: $anime_standart;
                                position: relative;
                                line-height: .7rem;
                                font-size: .6rem;

                                &:before {
                                    background: rgba(255, 255, 255, 0.04);
                                    border: 1px dashed #A5B1BF;
                                    transition: $anime_standart;
                                    border-radius: .2rem;
                                    position: absolute;
                                    display: block;
                                    opacity: 0.5;
                                    height: 100%;
                                    width: 100%;
                                    content: '';
                                    left: 0;
                                    top: 0;
                                }

                                &:hover {
                                    background-color: darken($color-success-bg, 25%);
                                    cursor: pointer;
                                }

                                &.check_circle {
                                    background-color: $color-success-bg;
                                    
                                    &:before {
                                        border: 1px solid $color-success;
                                    }
                                }

                                &.remove_circle {
                                    &:before {
                                        border: 1px solid $color-error;
                                    }
                                }

                                &__info {
                                    line-height: 152.8%;
                                    color: #7F8FA4;
                                }

                                &__btn {
                                    border: 1px solid $color-success;

                                    i {
                                        transform: rotateZ(45deg);
                                    }

                                    span {
                                        color: $color-success;
                                    }
                                }
                            }

                            &__list {
                                padding-top: .15rem;

                                .added_file {
                                    margin-top: .55rem;

                                    &__icon {
                                        background-color: $color-file-added;
                                        justify-content: center;
                                        border-radius: .25rem;
                                        align-items: center;
                                        position: relative;
                                        display: flex;

                                        i {
                                            color: $color-dark;
                                            font-size: 1.5rem;
                                        }

                                        span {
                                            padding-top: .5rem;
                                            position: absolute;
                                            font-size: .5rem;
                                            color: white;
                                        }
                                    }

                                    &__text {
                                        line-height: 0.731rem;
                                        color: $color-mischka;
                                        font-size: .6rem;
                                    }

                                    &__close {
                                        transition: $anime_standart;
                                        border-radius: .2rem;
                                        padding-left: .05rem;
                                        height: .7rem;
                                        width: .7rem;

                                        &:hover {
                                            background-color: $color-mischka;
                                            color: white;
                                        }
                                    }
                                }
                            }
                        }

                        &_link {
                            ::v-deep .v-input__control .v-input__slot {
                                padding: 0 1.2rem;
                                
                                .v-text-field__slot {
                                    font-size: .6rem;
                                }

                                fieldset {
                                    border-color: $color-border;
                                }

                                .v-input__append-inner {
                                    .v-input__icon {
                                        i {
                                            font-size: .9rem;
                                        }
                                    }
                                }
                            }

                            &.remove_circle {
                                ::v-deep .v-input__control .v-input__slot {
                                    fieldset {
                                        border-color: $color-error;
                                    }

                                    .v-input__append-inner i {
                                        caret-color:  $color-error !important;
                                        color: $color-error !important;
                                    }
                                }
                            }

                            &.check_circle {
                                ::v-deep .v-input__control .v-input__slot {
                                    fieldset {
                                        background-color: $color-success-bg;
                                        border-color: $color-success;
                                    }

                                    .v-input__append-inner i {
                                        caret-color: $color-success !important;
                                        color: $color-success !important;
                                    }
                                }
                            }
                        }

                        &_iframe {
                            ::v-deep .v-input__control {
                                .v-input__slot {
                                    margin-bottom: 0;
                                    padding: 1rem;

                                    fieldset {
                                        border-color: $color-border;
                                    }

                                    .v-text-field__slot textarea {
                                        line-height: .7rem;
                                        font-size: .6rem;
                                    }
                                }

                                .v-text-field__details {
                                    margin-top: .5rem;

                                    .v-messages .v-messages__wrapper .v-messages__message {
                                        line-height: .703rem;
                                        font-size: .6rem;
                                    }
                                }
                            }

                            &.remove_circle {
                                ::v-deep .v-input__control .v-input__slot {
                                    fieldset {
                                        border-color: $color-error;
                                    }

                                    .v-input__append-inner .v-input__icon i {
                                        caret-color:  $color-error !important;
                                        color: $color-error !important;
                                    }
                                }
                            }

                            &.check_circle {
                                ::v-deep .v-input__control .v-input__slot {
                                    fieldset {
                                        background-color: $color-success-bg;
                                        border-color: $color-success;
                                    } 

                                    .v-input__append-inner i {
                                        caret-color: $color-success !important;
                                        color: $color-success !important;
                                    }
                                }
                            }
                        }
                    }
                }
            }
            
            &_btn {
                margin-top: 3.5rem;

                .v-btn {
                    padding: 0 1.6rem;

                    &.full {
                        width: 100%;
                    }

                    ::v-deep .v-btn__content {
                        text-transform: none;
                        line-height: .8rem;
                        letter-spacing: 0;
                        font-weight: 400;
                        font-size: .8rem;
                    }
                }
            }
        }
    }
</style>