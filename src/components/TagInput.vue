<template>
    <div class="control">
        <label for='tags' class="label" v-if="label" v-text="label"></label>
        <div class="input is-flex">
            <div class="tags">
                <div v-for="tag in tags"
                     class="tag is-outlined"
                     :class="[tag.selected ? 'is-warning' : 'is-primary']"
                     @click="toggleSelect(tag)"
                >
                    {{ tag.name }}
                    <button class="delete is-small" @click.prevent="removeTag(tag)"></button>
                </div>
            </div>
            <input type="text" class="invisible-input flex" :placeholder="placeholder" @keydown="onKeyDown" v-model="inputBuffer">
        </div>
        <input type="hidden" name="tags[]" v-for="tag in tags" :value="tag.name">
    </div>
</template>

<style>
    .tag {
        border-radius: 5px;
        margin-right: 5px;
    }

    .invisible-input {
        border: none;
        outline: none;
    }
</style>

<script>
    export default {
        data() {
            return {
                tags: [],
                inputBuffer: ''
            }
        },

        props: {
            label: {
                type: String,
                default: null
            },

            placeholder: {
                type: String,
                default: null
            },

            maxNumberOfTags: {
                type: Number,
                default: 5
            }
        },

        computed: {

            /** @var Tag selectedTag */
            selectedTag() {
                if (!this.hasSelectedTag()) {
                    return null;
                }

                let tagIndex = this.tags.findIndex((tag) => { return tag.selected });

                return this.tags[tagIndex];
            },

            /** @var Tag lastTag */
            lastTag() {
                return this.tags[this.tags.length - 1];
            },

            /** @var Tag firstTag */
            firstTag() {
                return this.tags[0];
            }
        },

        methods: {

            /**
             * Adds a new tag to the list of tags
             *
             * @param string name
             */
            addTag(name) {
                if (this.tags.length == this.maxNumberOfTags) {
                    return;
                }

                if (!this.hasTag(name)) {
                    this.tags.push({
                        name: name,
                        selected: false
                    });
                }

                if (this.selectedTag) {
                    this.removeTag(this.selectedTag);
                }
            },

            /**
             * Checks if a tag with a specific name already exists
             *
             * @param string name
             *
             * @return bool
             */
            hasTag(name) {
                return this.tags.find((tag) => { return tag.name == name }) != undefined
            },

            /**
             * Removes a tag from the tags list
             *
             * @param Tag tag
             */
            removeTag(tag) {
                if (tag == null) {
                    return;
                }

                this.tags = this.tags.filter((item) => {
                    return item.name != tag.name
                });
            },

            /**
             * Toggles a tags selected state
             *
             * @param Tag tag
             */
            toggleSelect(tag) {
                if (tag.selected) {
                    return tag.selected = false;
                }

                this.deselectAllTags();

                tag.selected = true;
                this.inputBuffer = tag.name;
            },

            /**
             * Sets all tags to be not selected
             */
            deselectAllTags() {
                this.tags.forEach((tag) => { tag.selected = false });
            },

            /**
             * Checks if component has any selected tags
             *
             * @return bool
             */
            hasSelectedTag() {
                return this.tags.find((tag) => { return tag.selected }) != undefined;
            },

            /**
             * Reset/Clears the input buffer
             */
            clearBuffer() {
                this.inputBuffer = '';
            },

            /**
             * Key down event
             *
             * @param KeyboardEvent event
             */
            onKeyDown(event) {
                if (event.code == 'Space') {
                    return this.onSpace(event);
                }

                if (event.key == 'Backspace') {
                    return this.onBackspace(event);
                }

                if (event.key == 'ArrowLeft') {
                    return this.onArrowLeft(event);
                }

                if (event.key == 'ArrowRight') {
                    return this.onArrowRight(event);
                }
            },

            /**
             * Handle Space key
             *
             * @param KeyboardEvent event
             */
            onSpace(event) {
                event.preventDefault();

                if (this.inputBuffer != '') {
                    this.addTag(this.inputBuffer);
                    this.clearBuffer();
                }
            },

            /**
             * Handle Backspace key
             *
             * @param KeyboardEvent event
             */
            onBackspace(event) {
                if (this.inputBuffer == '' && this.tags.length <= 0) {
                    return;
                }

                if (this.inputBuffer != '') {
                    return;
                }

                event.preventDefault();

                if (this.selectedTag) {
                    this.removeTag(this.selectedTag);
                    return this.deselectAllTags();
                }

                this.lastTag.selected = true;
                this.inputBuffer = this.selectedTag.name;
            },

            /**
             * Handle Left Arrow key
             *
             * @param KeyboardEvent event
             */
            onArrowLeft(event) {
                if (this.inputBuffer != '') {
                    return;
                }

                if (this.tags.length <= 0) {
                    return;
                }

                if (!this.selectedTag) {
                    return this.lastTag.selected = true;
                }

                let tagIndex = this.tags.findIndex((tag) => { return tag.name == this.selectedTag.name });

                if (tagIndex - 1 >= 0) {
                    this.deselectAllTags();
                    this.tags[tagIndex - 1].selected = true;
                }
            },

            /**
             * Handle Right Arrow key
             *
             * @param KeyboardEvent event
             */
            onArrowRight(event) {
                if (this.inputBuffer != '') {
                    return;
                }

                if (this.tags.length <= 0) {
                    return;
                }

                if (!this.selectedTag) {
                    return this.firstTag.selected = true;
                }

                let tagIndex = this.tags.findIndex((tag) => { return tag.name == this.selectedTag.name });

                if (tagIndex + 1 < this.tags.length) {
                    this.deselectAllTags();
                    this.tags[tagIndex + 1].selected = true;
                }
            }
        }
    }
</script>
