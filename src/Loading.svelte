<script>
  export let loading;
</script>

{#if loading}
  <div class="spectrum-ProgressCircle spectrum-ProgressCircle--indeterminate">
    <div class="spectrum-ProgressCircle-track" />
    <div class="spectrum-ProgressCircle-fills">
      <div class="spectrum-ProgressCircle-fillMask1">
        <div class="spectrum-ProgressCircle-fillSubMask1">
          <div class="spectrum-ProgressCircle-fill" />
        </div>
      </div>
      <div class="spectrum-ProgressCircle-fillMask2">
        <div class="spectrum-ProgressCircle-fillSubMask2">
          <div class="spectrum-ProgressCircle-fill" />
        </div>
      </div>
    </div>
  </div>
{/if}
