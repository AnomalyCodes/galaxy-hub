<template>
    <div :class="['redirect', 'alert', 'alert-warning', cancelled ? 'cancelled' : 'active']">
        <p>
            {{ preText }}<a :href="parsedUrl">{{ url }}</a
            >{{ postText }}
        </p>
        <p class="button-line">
            You will be redirected in {{ delay }} seconds.
            <button @click="cancel" class="btn btn-light">Cancel</button>
            <span v-if="cancelled">Redirect cancelled!</span>
        </p>
    </div>
</template>

<script>
import { repr, doRedirect } from "~/utils.js";
export default {
    props: {
        location: { type: Object, required: true },
        preText: { type: String, required: false, default: "This content has a new home at " },
        url: { type: String, required: true },
        postText: { type: String, required: false, default: "." },
        delay: { type: Number, required: false, default: 5 },
    },
    data() {
        return {
            cancelled: false,
        };
    },
    computed: {
        // Need to set this here to make sure the component will update when
        // `url` or `location` does.
        parsedUrl() {
            return parseUrl(this.url, this.location);
        },
    },
    methods: {
        cancel() {
            console.log("Cancelling redirect..");
            this.cancelled = true;
        },
    },
    mounted() {
        redirectIfPossible(this);
    },
    beforeUpdate() {
        redirectIfPossible(this);
    },
};
function redirectIfPossible(data) {
    if (data.location && data.url) {
        console.log(repr`Redirecting to ${data.url} in ${data.delay} seconds..`);
        let currentPath = data.location.pathname;
        setTimeout(() => doRedirect(data.url, currentPath, data.cancelled), data.delay * 1000);
    }
}
function parseUrl(rawUrl, location) {
    if (location === undefined) {
        return undefined;
    } else if (rawUrl.startsWith("/")) {
        return `${location.origin}${rawUrl}`;
    } else if (rawUrl.startsWith("http://") || rawUrl.startsWith("https://")) {
        return rawUrl;
    } else {
        console.error(repr`Unrecognized redirect url ${rawUrl}`);
    }
}
</script>

<style scoped>
.redirect {
    margin-top: 20px;
}
p {
    margin-bottom: 10px;
}
p.button-line {
    margin-bottom: 0;
}
button {
    margin: 0 10px;
}
</style>
