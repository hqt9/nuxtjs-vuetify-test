<template>
    <div class="ma-8">
        <button @click="count++">You clicked me {{ count }} times.</button>

        <div class="py-16">
            <p>{{ total }}</p>
            <button-counter
                v-on:increment="incrementTotal"
                class="pr-2"
            ></button-counter>
            <button-counter v-on:increment="incrementTotal"></button-counter>
        </div>
    </div>
</template>

<script>
import ButtonCounter from '../components/ButtonCounter'

export default {
    components: { ButtonCounter },
    data() {
        return {
            count: 0,
            total: 0
        }
    },
    methods: {
        incrementTotal() {
            this.total++
        }
    }
}
</script>
