<script>
  import { getContext, onDestroy } from "svelte";
  import Captures from "./Captures.svelte";
  import Loading from "./Loading.svelte";

  export let field;
  export let label;

  const { API, styleable } = getContext("sdk");
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  let loading = false;
  let previewEnabled = false;

  let mirrored = true; // make this smarter mobile vs desktop?
  let imageCaptured = false;
  let videoSource = {};

  let fieldApi;
  let fieldState;

  let blob;
  let imageCapture;
  let imageUrl;

  const formApi = formContext?.formApi;
  const labelPos = fieldGroupContext?.labelPosition || "above";
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(
    field,
    "attachment",
    [],
    false,
    null,
    formStep
  );

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  const openPreview = async () => {
    loading = true;
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: true,
      });
      const track = stream.getVideoTracks()[0];
      imageCapture = new ImageCapture(track);

      videoSource.srcObject = stream;
      // Fix for iOS Safari from https://leemartin.dev/hello-webrtc-on-safari-11-e8bcb5335295
      videoSource.setAttribute("autoplay", "");
      videoSource.setAttribute("muted", "");
      videoSource.setAttribute("playsinline", "");
      videoSource.play();
      videoSource = videoSource;

      imageCapture.takePhoto();
      previewEnabled = true;
    } catch (error) {
      console.log(error);
      previewEnabled = false;
    }
    loading = false;
  };

  const capture = async () => {
    blob = await imageCapture.takePhoto();
    imageCaptured = true;
    blob.name = "capture.jpg";
    blob.extension = "jpeg";
    imageUrl = URL.createObjectURL(blob);
  };

  const upload = async () => {
    const res = await uploadImage(blob);
    imageUrl = res[0].url;

    const files = fieldState?.value || [];
    files.push(res[0]);
    fieldApi?.setValue(files);
    imageCaptured = false;
    openPreview();
  };

  const uploadImage = async (image) => {
    let data = new FormData();
    data.append("file", image, "capture.jpg");
    imageCaptured = false;
    try {
      const res = await API.uploadAttachment({
        data,
        tableId: formContext?.dataSource?.tableId,
      });
      return res;
    } catch (error) {
      return [];
    }
  };

  const remove = () => {
    blob = null;
    imageCaptured = false;
    imageUrl = null;
    openPreview();
  };

  const stop = () => {
    var stream = videoSource.srcObject;
    var tracks = stream.getTracks();

    for (var i = 0; i < tracks.length; i++) {
      var track = tracks[i];
      track.stop();
    }

    videoSource = {};
    imageCaptured = false;
    previewEnabled = false;
  };
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form</div>
  {:else}
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>
    <div class="spectrum-Form-itemField">
      {#if !field}
        <div class="error">Please select a field</div>
      {:else}
        <div class="container">
          <Loading {loading} />

          {#if previewEnabled || loading}
            {#if imageCaptured}
              <canvas id="takePhotoCanvas" />
              <img alt="preview" src={imageUrl} />
              <div class="actions">
                <button
                  on:click={remove}
                  class="spectrum-Button spectrum-Button--fill spectrum-Button--sizeM spectrum-Button--primary"
                  >Remove</button
                >
                <button
                  on:click={upload}
                  class="spectrum-Button spectrum-Button--fill spectrum-Button--sizeM spectrum-Button--primary"
                  >Use photo</button
                >
              </div>
            {:else}
              <div>
                <!-- svelte-ignore a11y-media-has-caption -->
                <video class:mirrored bind:this={videoSource} />
              </div>
            {/if}
          {/if}

          {#if !imageCaptured}
            <div class="actions">
              {#if previewEnabled}
                <button
                  on:click={stop}
                  class="spectrum-ActionButton spectrum-ActionButton--sizeL"
                >
                  <svg
                    class="spectrum-Icon spectrum-Icon--sizeL spectrum-ActionButton-icon"
                    focusable="false"
                  >
                    <use xlink:href="#spectrum-icon-18-Close" />
                  </svg>
                </button>

                <button
                  on:click={capture}
                  class="spectrum-ActionButton spectrum-ActionButton--sizeL"
                >
                  <svg
                    class="spectrum-Icon spectrum-Icon--sizeL spectrum-ActionButton-icon"
                    focusable="false"
                  >
                    <use xlink:href="#spectrum-icon-18-Camera" />
                  </svg>
                </button>
                <button
                  on:click={() => (mirrored = !mirrored)}
                  class="spectrum-ActionButton spectrum-ActionButton--sizeL"
                >
                  <svg
                    class="spectrum-Icon spectrum-Icon--sizeL spectrum-ActionButton-icon"
                    focusable="false"
                  >
                    <use xlink:href="#spectrum-icon-18-FlipHorizontal" />
                  </svg>
                </button>
              {:else}
                <button
                  on:click={openPreview}
                  class="spectrum-Button spectrum-Button--fill spectrum-Button--sizeM spectrum-Button--primary"
                  >Open camera</button
                >
              {/if}
            </div>
          {/if}

          <Captures attachments={fieldState?.value} />
        </div>

        {#if fieldState?.error}
          <div class="error">{fieldState.error}</div>
        {/if}
      {/if}
    </div>
  {/if}
</div>

<style>
  img {
    width: 100%;
  }
  video {
    width: 100%;
  }
  canvas {
    width: 100%;
    display: none;
  }

  .actions {
    display: flex;
    justify-content: space-between;
  }

  .mirrored {
    transform: rotateY(180deg);
    -webkit-transform: rotateY(180deg);
    -moz-transform: rotateY(180deg);
  }

  .spectrum-ActionButton .spectrum-Icon {
    margin-left: calc(
      -1 * (var(--spectrum-actionbutton-textonly-padding-left-adjusted) -
            var(--spectrum-actionbutton-icononly-padding-left-adjusted))
    ) !important;
  }
</style>
