<template>

    <div>

        <b-form v-if="model" @submit="save">

            <yc-admin-buttons :model="model" @save="save" @destroy="destroy"></yc-admin-buttons>

            <b-card
                class="mb-4"
                header="Общая информация"
                header-class="text-white bg-secondary"
                no-body
            >

                <b-card-body>

                    <b-form-group
                        label="Название"
                        label-for="title"
                        label-cols-sm="2"
                    >
                        <b-form-input
                            id="title"
                            type="text"
                            v-model="model.title"
                            required
                            trim />
                    </b-form-group>

                    <b-form-group
                        label="Системное название"
                        label-for="name"
                        label-cols-sm="2"
                    >
                        <b-form-input
                            id="name"
                            type="text"
                            v-model="model.name"
                            required
                            trim />
                    </b-form-group>

                    <b-form-group
                        label="Ширина"
                        label-for="width"
                        label-cols-sm="2"
                    >
                        <b-form-input
                            id="width"
                            type="number"
                            min="1" max="1900"
                            v-model="model.width"
                            required
                            trim />
                    </b-form-group>

                    <b-form-group
                        label="Высота"
                        label-for="height"
                        label-cols-sm="2"
                    >
                        <b-form-input
                            id="height"
                            type="number"
                            min="1" max="1200"
                            v-model="model.height"
                            required
                            trim />
                    </b-form-group>

                    <b-form-group
                        label="Качество"
                        label-for="quality"
                        label-cols-sm="2"
                    >
                        <b-form-input
                            id="quantity"
                            type="number"
                            min="1" max="100"
                            v-model="model.quality"
                            required
                            trim />
                    </b-form-group>

                    <b-form-group
                        label="Экшен"
                        label-for="action"
                        label-cols-sm="2"
                    >
                        <b-form-select
                            id="action"
                            class="col-3"
                            v-model="model.action"
                            :options="actions">
                        </b-form-select>
                        <small tabindex="-1" class="form-text text-muted">
                            Настройка типа изменение размера изображения<br>
                            <b>Resize</b> - пропорционально изменяет размер изображения без обрезки<br>
                            <b>Thumbnail Inset</b> - вписывает изображение в заданный размер без обрезки, центрирует и добавлят белый фон<br>
                            <b>Thumbnail Outbound</b> - обрезает и вписывает изображение в заданный размер
                        </small>
                    </b-form-group>

                    <b-form-group
                        label="Копирайт"
                        label-for="watermark"
                        label-cols-sm="2"
                        description="Имя файла в папке storage/uploads"
                    >
                        <b-form-input
                            id="watermark"
                            type="text"
                            v-model="model.watermark"
                            trim />
                    </b-form-group>

                    <b-form-checkbox
                        id="isDefault"
                        v-model="model.isDefault"
                        name="isDefault"
                        :value="1"
                        :unchecked-value="0"
                    >
                        Использовать по умолчанию
                    </b-form-checkbox>

                </b-card-body>

            </b-card>

            <b-button type="submit" variant="primary" :disabled="isLoading">Сохранить</b-button>

            <pre v-if="isDev">model: {{  model }}</pre>

        </b-form>

    </div>

</template>

<script>

    export default {

        computed: {
            isDev: function () {
                return this.$store.getters['commerce/isDev'];
            },
            isLoading: function () {
                return this.$store.getters['commerce/isLoading'];
            },
            hasError: function () {
                return this.$store.getters['commerce/hasError'];
            },
            model: function () {
                return this.$store.getters['files-presets/model'];
            },
            settings () {
                return this.$store.getters['commerce/settings'];
            },
            actions () {
                return _.isEmpty(this.settings) ? [] : this.settings.files.presets.actions;
            }
        },

        created () {
            this.$store.dispatch('files-presets/find', this.$route.query.id);
        },

        watch: {
            '$route': function () {
                this.$store.dispatch('files-presets/find', this.$route.query.id);
            }
        },

        methods: {
            save (event) {
                event.preventDefault();

                let isNewRecord = (this.model.id === null);

                this.$store.dispatch('files-presets/save', this.model).then(() => {
                    if (this.hasError) {
                        return false;
                    }

                    this.$notify({type: 'success', text: 'Пресет сохранен'});

                    if (isNewRecord) {
                        this.$router.push({ path: `/files/preset/update?id=${this.model.id}` });
                    }
                });

            },

            destroy () {
                this.$store.dispatch('files-presets/delete', this.model.id).then(() => {
                    this.$notify({type: 'success', text: 'Пресет удален'});
                    this.$store.dispatch('commerce/fetchSettings');
                    this.$router.push({ path: '/files/preset/index' });
                });
            }
        }

    }
</script>
