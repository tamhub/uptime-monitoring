<template>
    <span class="badge rounded-pill" :style=" 'background-color: ' + color ">{{ text }}</span>
</template>

<script>
export default {
    props: {
        /** Current status of monitor */
        status: {
            type: Number,
            default: 0,
        }
    },

    computed: {
        color() {
            if (this.status === 0) {
                return "#dc3545";
            }

            if (this.status === 1) {
                return "#5cdd8b";
            }

            if (this.status === 2) {
                return "#f8a306";
            }

            if (this.status === 3) {
                return "#1747f5";
            }

            return "#584fff";
        },

        text() {
            if (this.status === 0) {
                return this.$t("Down");
            }

            if (this.status === 1) {
                return this.$t("Up");
            }

            if (this.status === 2) {
                return this.$t("Pending");
            }

            if (this.status === 3) {
                return this.$t("statusMaintenance");
            }

            return this.$t("Unknown");
        },
    },
};
</script>

<style scoped>
    span {
        min-width: 64px;
    }
</style>
