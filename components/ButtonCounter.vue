<template>
    <button @click="increment">{{ counter }}</button>
</template>

<script>
export default {
    props: {},
    data: function () {
        return {
            counter: 0
        }
    },
    methods: {
        increment() {
            this.counter += 1
            this.$emit('increment')
        }
    }
}
</script>