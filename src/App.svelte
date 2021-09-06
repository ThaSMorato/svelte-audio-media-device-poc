<script>

  const DOMSVars = {
    audiosBlobsUrls : [],
    isRecording : false,
    mediaAudioStream: false
  }

  let id = 0;

  const DOMSFunctions = {
    start: () => {return;},
    stop: () => {return;}
  }

  const getAudioMediaStream = async () => {
    return await navigator.mediaDevices.getUserMedia({audio : true});
  }

  const handleMediaRecorder = (stream) => {
    let mediaRecorder = new MediaRecorder(stream);

    DOMSVars.mediaAudioStream = true;

    let chunks = [];
    
    mediaRecorder.ondataavailable = (e) => {
      chunks = [...chunks, e.data];
    }


    mediaRecorder.onstop = (e) => {
      const blob = new Blob(chunks, { 'type' : 'audio/ogg; codecs=opus' });
      chunks = [];

      const audioURL = URL.createObjectURL(blob);
      const audioObject = {
        id: id++,
        url : audioURL,
      }

      DOMSVars.audiosBlobsUrls = [...DOMSVars.audiosBlobsUrls, audioObject];
    }

    DOMSFunctions.start = () => { 
      DOMSVars.isRecording = true;
      mediaRecorder.start();
    }

    

    DOMSFunctions.stop = () => {
      DOMSVars.isRecording = false;
      mediaRecorder.stop();
    }
  }

  const clearUrl = (id) => {
    DOMSVars.audiosBlobsUrls = DOMSVars.audiosBlobsUrls.filter(audio => audio.id != id);
  }

  ( async () => {
    const audioMedia = await getAudioMediaStream();
    return handleMediaRecorder(audioMedia);
  })()

</script>

<main>
  <h1>Gravador de áudio</h1>
  <div class="buttons">
    <button class="record" on:click={DOMSFunctions.start} disabled={DOMSVars.isRecording || !DOMSVars.mediaAudioStream}>
      Record
    </button>
    <button class="pause" on:click={DOMSFunctions.stop} disabled={!DOMSVars.isRecording || !DOMSVars.mediaAudioStream}>
      Pause
    </button>
    <span class={DOMSVars.isRecording && "recording"}>
      {DOMSVars.isRecording? "Gravando" : "Aguardando..."}
    </span>
  </div>
  
  <div class="audio__wrapper">
    {#each DOMSVars.audiosBlobsUrls as {url, id}}
      <div class="audio">
        <span>{id}</span>
        <audio src={url} controls>
          O seu navegador não suporta o elemento <code>audio</code>.
        </audio>

        <button class="clear" on:click={() => clearUrl(id)}> 
          X
        </button>
      </div>
    {/each}
  </div>
</main>

<style>
  main {
    width: 100vw;
    height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    padding: 20px 0;
    background: white;
  }

  .audio {
    height: 50px;
    background: #f1f3f4;
    display: flex;
    align-items: center;
    justify-content: space-evenly;
    width: 350px;
    border-radius: 25px;
    border: 1px solid gray;
    margin: 10px 0;
    padding: 2px;
  }

  audio {
    color: white;
  }

  span {
    margin: 0 10px;
    color: black;
    font-weight: 900;
  } 

  button {
    cursor: pointer;
  }

  .clear {
    background: none;
    color: black;
    border: none;
    margin: 0 10px;
    cursor: pointer;
    font-weight: 900;
  }

  .record:disabled {
    color: white;
    border-color: white;
    background-color: red;
  }

  .recording {
    color: red;
  }
</style>