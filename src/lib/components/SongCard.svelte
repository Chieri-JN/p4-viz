<script lang="ts">
    let { embedUrl }: { embedUrl: string } = $props();
</script>

<div class="song-card">
    {@html embedUrl}
</div>

<style>
    .song-card {
        width: 100%;
        border-radius: 10px;
        overflow: hidden;
    }

    .song-card :global(iframe) {
        width: 100% !important;
        display: block;
        border: 0;
        border-radius: 10px;
    }
</style>
