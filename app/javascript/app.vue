<template>
    <draggable v-model="lists" :options="{groups: 'lists'}" class="board dragArea" @end="listMoved">
        <list v-for="(list, index) in lists" :list="list"></list>
        <div class="list">
            <a v-if="!editing" v-on:click="startEditing">Add a List</a>
            <textarea v-if="editing" v-model="message" ref="message" class="form-control mb-1"></textarea>
            <button v-if="editing" v-on:click="createList" class="btn btn-secondary">Add</button>
            <a v-if="editing" v-on:click="editable=false">Cancel</a>
        </div>
    </draggable>

</template>

<script>
    import draggable from 'vuedraggable'
    import list from 'components/list'

    export default {
        components: {draggable, list},

        data: function () {
            return {
                editing: false,
                message: "",
            }
        },

        computed: {
            lists(){
                return this.$store.state.lists;
            }
        },

        methods: {
            startEditing: function () {
                this.editing = true
                this.$nextTick(() => {
                    this.$refs.message.focus()
                })
            },
            listMoved: function (event) {
                var data = new FormData
                data.append("list[position]", event.newIndex + 1)
                Rails.ajax({
                    beforeSend: () => true,
                    url: `/lists/${this.lists[event.newIndex].id}/move`,
                    type: "PATCH",
                    data: data,
                    dataType: "json",
                })
            },

            createList: function () {
                var data = new FormData
                data.append("list[name]", this.message)

                Rails.ajax({
                    beforeSend: () => true,
                    url: `/lists`,
                    type: "POST",
                    data: data,
                    dataType: "json",
                    success: (data) => {
                        window.store.commit('addList', data);
                        this.message = ""
                        this.editing = false
                    }
                })
            },

        }
    }
</script>
<style scoped>
    .board {
        overflow-x: auto;
        white-space: nowrap;
    }

    .dragArea {
        min-height: 10px;
    }

    .list {
        display: inline-block;
        border-radius: 3px;
        padding: 20px;
        background: #E2E4E6;
        width: 270px;
        vertical-align: top;
        margin-right: 56px;
    }
</style>
