```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Multi Track Audio Player</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>

<style>
*{
    box-sizing:border-box;
}

body{
    margin:0;
    font-family:Arial,Helvetica,sans-serif;
    background:#080b12;
    color:#fff;
    min-height:100vh;
}

.container{
    width:min(1200px,94%);
    margin:auto;
    padding:20px 0 40px;
}

h1{
    text-align:center;
    margin:5px 0 20px;
    font-size:28px;
}

.top-buttons{
    display:flex;
    justify-content:center;
    gap:10px;
    flex-wrap:wrap;
    margin-bottom:18px;
}

button,
.file-label{
    border:0;
    padding:12px 18px;
    border-radius:10px;
    background:#171e2d;
    color:#fff;
    cursor:pointer;
    font-size:14px;
    font-weight:bold;
    border:1px solid #303b52;
    transition:.2s;
}

button:hover,
.file-label:hover{
    background:#25314a;
}

input[type="file"]{
    display:none;
}

.master{
    background:#101621;
    border:1px solid #273247;
    border-radius:16px;
    padding:18px;
    margin-bottom:20px;
}

.master-controls{
    display:flex;
    justify-content:center;
    gap:10px;
    flex-wrap:wrap;
    margin-bottom:15px;
}

#masterSeekbar{
    width:100%;
    cursor:pointer;
}

.time-row{
    display:flex;
    justify-content:space-between;
    font-size:13px;
    color:#aeb8ca;
    margin-top:5px;
}

.tracks{
    display:grid;
    grid-template-columns:repeat(auto-fit,minmax(280px,1fr));
    gap:14px;
}

.track{
    background:#101621;
    border:1px solid #273247;
    border-radius:15px;
    padding:15px;
    min-width:0;
}

.track-header{
    display:flex;
    align-items:center;
    justify-content:space-between;
    gap:8px;
    margin-bottom:8px;
}

.track-number{
    font-weight:bold;
    font-size:17px;
}

.status{
    font-size:11px;
    padding:5px 8px;
    border-radius:20px;
    background:#263149;
    color:#b9c5d8;
}

.status.loaded{
    background:#123b2c;
    color:#5ff0a7;
}

.status.playing{
    background:#183b63;
    color:#66b8ff;
}

.file-name{
    font-size:13px;
    color:#c6cfdd;
    overflow:hidden;
    text-overflow:ellipsis;
    white-space:nowrap;
    margin-bottom:13px;
}

.channel-box{
    display:flex;
    align-items:center;
    gap:16px;
    background:#0a0e16;
    border:1px solid #222c3e;
    padding:10px 12px;
    border-radius:10px;
    margin-bottom:13px;
}

.channel-title{
    color:#8e9bb0;
    font-size:12px;
    margin-right:auto;
}

.channel-option{
    display:flex;
    align-items:center;
    gap:5px;
    font-size:13px;
    cursor:pointer;
    user-select:none;
}

.channel-option input{
    width:18px;
    height:18px;
    cursor:pointer;
    accent-color:#38a8ff;
}

.volume-row{
    display:flex;
    align-items:center;
    gap:10px;
}

.volume-label{
    font-size:12px;
    color:#8e9bb0;
    width:55px;
}

.volume-slider{
    flex:1;
    cursor:pointer;
}

.volume-value{
    width:42px;
    text-align:right;
    font-size:12px;
    color:#b7c2d4;
}

.empty{
    text-align:center;
    color:#68758b;
    padding:45px 10px;
    border:1px dashed #303b4f;
    border-radius:15px;
}

.loading{
    display:none;
    position:fixed;
    inset:0;
    background:rgba(0,0,0,.72);
    z-index:1000;
    align-items:center;
    justify-content:center;
    flex-direction:column;
    gap:12px;
}

.loading.show{
    display:flex;
}

.spinner{
    width:42px;
    height:42px;
    border:4px solid #263248;
    border-top-color:#55b9ff;
    border-radius:50%;
    animation:spin .8s linear infinite;
}

@keyframes spin{
    to{transform:rotate(360deg);}
}

.info{
    text-align:center;
    color:#76849a;
    font-size:12px;
    margin-top:18px;
}

@media(max-width:600px){
    h1{
        font-size:23px;
    }

    .container{
        width:95%;
        padding-top:12px;
    }

    .tracks{
        grid-template-columns:1fr;
    }

    button,
    .file-label{
        width:100%;
        text-align:center;
    }

    .top-buttons{
        flex-direction:column;
    }
}
</style>
</head>

<body>

<div class="loading" id="loading">
    <div class="spinner"></div>
    <div id="loadingText">Loading...</div>
</div>

<div class="container">

    <h1>🎵 Multi Track Audio Player</h1>

    <div class="top-buttons">

        <label class="file-label">
            📁 Select Folder
            <input
                type="file"
                id="folderInput"
                webkitdirectory
                directory
                multiple
            >
        </label>

        <label class="file-label">
            🎵 Select Files / ZIP
            <input
                type="file"
                id="fileInput"
                multiple
                accept="audio/*,.wav,.mp3,.ogg,.flac,.aac,.m4a,.webm,.opus,.zip"
            >
        </label>

        <button id="resetBtn">🔄 New Load / Reset</button>

    </div>

    <div class="master">

        <div class="master-controls">

            <button id="playAllBtn">
                ▶ Play All
            </button>

            <button id="pauseAllBtn">
                ⏸ Pause All
            </button>

        </div>

        <input
            type="range"
            id="masterSeekbar"
            min="0"
            max="0"
            value="0"
            step="0.01"
            disabled
        >

        <div class="time-row">
            <span id="currentTime">00:00</span>
            <span id="totalTime">00:00</span>
        </div>

    </div>

    <div id="tracksContainer" class="tracks">

        <div class="empty">
            Select a folder, audio files, or ZIP file to create players.
        </div>

    </div>

    <div class="info">
        Left / Right channel controls are independent for every player.
    </div>

</div>


<script>

const AUDIO_EXTENSIONS = [
    "wav",
    "mp3",
    "ogg",
    "flac",
    "aac",
    "m4a",
    "webm",
    "opus"
];

let tracks = [];

let audioContext = null;

let isPlayingAll = false;

let masterUpdating = false;


/* =========================================================
   AUDIO CONTEXT
========================================================= */

function getAudioContext(){

    if(!audioContext){

        const AudioContext =
            window.AudioContext ||
            window.webkitAudioContext;

        if(!AudioContext){
            alert("Web Audio API is not supported in this browser.");
            return null;
        }

        audioContext = new AudioContext();
    }

    if(audioContext.state === "suspended"){
        audioContext.resume();
    }

    return audioContext;
}


/* =========================================================
   FILE CHECK
========================================================= */

function isAudioFile(file){

    const name = file.name.toLowerCase();

    const ext =
        name.includes(".")
        ? name.split(".").pop()
        : "";

    return AUDIO_EXTENSIONS.includes(ext);
}


function isZipFile(file){

    const name = file.name.toLowerCase();

    return (
        name.endsWith(".zip") ||
        file.type === "application/zip" ||
        file.type === "application/x-zip-compressed"
    );
}


/* =========================================================
   LOADING UI
========================================================= */

function showLoading(text){

    document.getElementById("loadingText").textContent = text;

    document.getElementById("loading").classList.add("show");
}


function hideLoading(){

    document.getElementById("loading").classList.remove("show");
}


/* =========================================================
   FORMAT TIME
========================================================= */

function formatTime(seconds){

    if(!isFinite(seconds) || seconds < 0){
        return "00:00";
    }

    const hours = Math.floor(seconds / 3600);

    const minutes =
        Math.floor((seconds % 3600) / 60);

    const secs =
        Math.floor(seconds % 60);

    if(hours > 0){

        return (
            String(hours).padStart(2,"0") +
            ":" +
            String(minutes).padStart(2,"0") +
            ":" +
            String(secs).padStart(2,"0")
        );

    }

    return (
        String(minutes).padStart(2,"0") +
        ":" +
        String(secs).padStart(2,"0")
    );
}


/* =========================================================
   CLEAN OLD TRACKS
========================================================= */

function clearExistingTracks(){

    tracks.forEach(track => {

        try{
            track.audio.pause();
        }catch(e){}

        try{
            if(track.sourceNode){
                track.sourceNode.disconnect();
            }

            if(track.splitter){
                track.splitter.disconnect();
            }

            if(track.leftGain){
                track.leftGain.disconnect();
            }

            if(track.rightGain){
                track.rightGain.disconnect();
            }

            if(track.merger){
                track.merger.disconnect();
            }
        }catch(e){}

        try{
            URL.revokeObjectURL(track.objectURL);
        }catch(e){}

    });

    tracks = [];

    isPlayingAll = false;

    document.getElementById("tracksContainer").innerHTML = `
        <div class="empty">
            Select a folder, audio files, or ZIP file to create players.
        </div>
    `;

    document.getElementById("masterSeekbar").value = 0;
    document.getElementById("masterSeekbar").max = 0;
    document.getElementById("masterSeekbar").disabled = true;

    document.getElementById("currentTime").textContent = "00:00";
    document.getElementById("totalTime").textContent = "00:00";
}


/* =========================================================
   CREATE EXACT STEREO CHANNEL ROUTING
========================================================= */

function setupChannelRouting(track, index){

    const ctx = getAudioContext();

    if(!ctx){
        return;
    }

    /*
        IMPORTANT:

        Audio
          ↓
        ChannelSplitter
          ↓
        LEFT  → Left Gain  ┐
                           ├→ ChannelMerger → Destination
        RIGHT → Right Gain ┘

        This keeps Left and Right physically separated.
    */

    const source =
        ctx.createMediaElementSource(track.audio);

    const splitter =
        ctx.createChannelSplitter(2);

    const leftGain =
        ctx.createGain();

    const rightGain =
        ctx.createGain();

    const merger =
        ctx.createChannelMerger(2);


    source.connect(splitter);


    /*
        LEFT CHANNEL
    */

    splitter.connect(
        leftGain,
        0
    );

    leftGain.connect(
        merger,
        0,
        0
    );


    /*
        RIGHT CHANNEL
    */

    splitter.connect(
        rightGain,
        1
    );

    rightGain.connect(
        merger,
        0,
        1
    );


    /*
        OUTPUT
    */

    merger.connect(ctx.destination);


    /*
        BOTH CHANNELS ON INITIALLY
    */

    leftGain.gain.value = 1;

    rightGain.gain.value = 1;


    track.sourceNode = source;

    track.splitter = splitter;

    track.leftGain = leftGain;

    track.rightGain = rightGain;

    track.merger = merger;

    track.audioContext = ctx;
}


/* =========================================================
   CHANNEL CONTROL
========================================================= */

function updateChannelRouting(index){

    const track = tracks[index];

    if(!track){
        return;
    }

    const card =
        document.querySelector(
            `.track[data-index="${index}"]`
        );

    if(!card){
        return;
    }

    const left =
        card.querySelector(".left-check");

    const right =
        card.querySelector(".right-check");


    /*
        NEVER allow both unchecked.
    */

    if(!left.checked && !right.checked){

        /*
            Keep the checkbox that was just changed
            from making both channels OFF.

            We restore LEFT by default.
        */

        left.checked = true;
    }


    if(track.leftGain){

        track.leftGain.gain.setTargetAtTime(
            left.checked ? 1 : 0,
            audioContext.currentTime,
            0.005
        );
    }


    if(track.rightGain){

        track.rightGain.gain.setTargetAtTime(
            right.checked ? 1 : 0,
            audioContext.currentTime,
            0.005
        );
    }
}


/* =========================================================
   CREATE TRACK
========================================================= */

function createTrack(file,index){

    const audio = new Audio();

    audio.preload = "metadata";

    audio.volume = 0.8;

    audio.src = URL.createObjectURL(file);

    const track = {

        audio: audio,

        fileName: file.name,

        objectURL: audio.src,

        isLoaded: false,

        sourceNode: null,

        splitter: null,

        leftGain: null,

        rightGain: null,

        merger: null,

        audioContext: null
    };


    tracks.push(track);


    createTrackCard(
        index,
        file.name
    );


    /*
        Metadata loaded
    */

    audio.addEventListener(
        "loadedmetadata",
        () => {

            track.isLoaded = true;

            const card =
                document.querySelector(
                    `.track[data-index="${index}"]`
                );

            if(card){

                const status =
                    card.querySelector(".status");

                status.textContent = "Loaded";

                status.classList.add("loaded");
            }

            updateMaxDuration();

            checkPlayButtonState();
        }
    );


    /*
        PLAYING
    */

    audio.addEventListener(
        "play",
        () => {

            const card =
                document.querySelector(
                    `.track[data-index="${index}"]`
                );

            if(card){

                const status =
                    card.querySelector(".status");

                status.textContent = "Playing";

                status.classList.remove("loaded");

                status.classList.add("playing");
            }
        }
    );


    /*
        PAUSED
    */

    audio.addEventListener(
        "pause",
        () => {

            if(audio.ended){
                return;
            }

            const card =
                document.querySelector(
                    `.track[data-index="${index}"]`
                );

            if(card){

                const status =
                    card.querySelector(".status");

                status.textContent = "Loaded";

                status.classList.remove("playing");

                status.classList.add("loaded");
            }
        }
    );


    /*
        END
    */

    audio.addEventListener(
        "ended",
        () => {

            const card =
                document.querySelector(
                    `.track[data-index="${index}"]`
                );

            if(card){

                const status =
                    card.querySelector(".status");

                status.textContent = "Ended";

                status.classList.remove("playing");

                status.classList.add("loaded");
            }

            checkAllEnded();
        }
    );
}


/* =========================================================
   CREATE TRACK CARD
========================================================= */

function createTrackCard(index,fileName){

    const container =
        document.getElementById("tracksContainer");


    if(index === 0){
        container.innerHTML = "";
    }


    const card =
        document.createElement("div");

    card.className = "track";

    card.dataset.index = index;


    card.innerHTML = `

        <div class="track-header">

            <div class="track-number">
                Track ${index + 1}
            </div>

            <div class="status">
                Loading
            </div>

        </div>


        <div
            class="file-name"
            title="${escapeHTML(fileName)}"
        >
            ${escapeHTML(fileName)}
        </div>


        <div class="channel-box">

            <span class="channel-title">
                Speaker
            </span>

            <label class="channel-option">

                <input
                    type="checkbox"
                    class="channel-check left-check"
                    checked
                >

                <span>Left</span>

            </label>


            <label class="channel-option">

                <input
                    type="checkbox"
                    class="channel-check right-check"
                    checked
                >

                <span>Right</span>

            </label>

        </div>


        <div class="volume-row">

            <span class="volume-label">
                Volume
            </span>

            <input
                type="range"
                class="volume-slider"
                min="0"
                max="1"
                step="0.01"
                value="0.8"
            >

            <span class="volume-value">
                80%
            </span>

        </div>

    `;


    container.appendChild(card);


    const leftCheck =
        card.querySelector(".left-check");

    const rightCheck =
        card.querySelector(".right-check");

    const volumeSlider =
        card.querySelector(".volume-slider");

    const volumeValue =
        card.querySelector(".volume-value");


    /*
        LEFT CHECKBOX
    */

    leftCheck.addEventListener(
        "change",
        () => {

            /*
                If RIGHT is already unchecked,
                LEFT cannot be unchecked.
            */

            if(
                !leftCheck.checked &&
                !rightCheck.checked
            ){

                leftCheck.checked = true;

                return;
            }

            updateChannelRouting(index);
        }
    );


    /*
        RIGHT CHECKBOX
    */

    rightCheck.addEventListener(
        "change",
        () => {

            /*
                If LEFT is already unchecked,
                RIGHT cannot be unchecked.
            */

            if(
                !leftCheck.checked &&
                !rightCheck.checked
            ){

                rightCheck.checked = true;

                return;
            }

            updateChannelRouting(index);
        }
    );


    /*
        VOLUME
    */

    volumeSlider.addEventListener(
        "input",
        () => {

            const value =
                Number(volumeSlider.value);

            if(tracks[index]){

                tracks[index].audio.volume =
                    value;
            }

            volumeValue.textContent =
                Math.round(value * 100) + "%";
        }
    );
}


/* =========================================================
   ESCAPE HTML
========================================================= */

function escapeHTML(value){

    return String(value)
        .replace(/&/g,"&amp;")
        .replace(/</g,"&lt;")
        .replace(/>/g,"&gt;")
        .replace(/"/g,"&quot;")
        .replace(/'/g,"&#039;");
}


/* =========================================================
   INITIALIZE AUDIO ROUTING AFTER FILE LOAD
========================================================= */

function initializeTrackRouting(){

    /*
        Web Audio nodes are created only after files
        are selected. The AudioContext may remain
        suspended until Play is pressed.
    */

    tracks.forEach(
        (track,index) => {

            if(!track.sourceNode){

                try{

                    setupChannelRouting(
                        track,
                        index
                    );

                }catch(error){

                    console.error(
                        "Audio routing error:",
                        error
                    );
                }
            }
        }
    );
}


/* =========================================================
   LOAD FOLDER
========================================================= */

async function loadFolderFiles(fileList){

    showLoading(
        "Reading folder..."
    );

    try{

        const normalAudio = [];

        const zipFiles = [];


        for(
            const file of Array.from(fileList)
        ){

            if(isAudioFile(file)){

                normalAudio.push(file);

            }else if(isZipFile(file)){

                zipFiles.push(file);
            }
        }


        const allAudioFiles = [
            ...normalAudio
        ];


        /*
            ZIP AUDIO EXTRACTION
        */

        for(
            const zipFile of zipFiles
        ){

            showLoading(
                "Opening " + zipFile.name + "..."
            );


            try{

                const zip =
                    await JSZip.loadAsync(zipFile);


                const entries =
                    Object.values(zip.files);


                for(
                    const entry of entries
                ){

                    if(entry.dir){
                        continue;
                    }


                    const entryName =
                        entry.name;


                    const fakeFileName =
                        entryName
                            .split("/")
                            .pop();


                    if(!isAudioFile({
                        name: fakeFileName
                    })){
                        continue;
                    }


                    const blob =
                        await entry.async("blob");


                    const file =
                        new File(
                            [blob],
                            fakeFileName,
                            {
                                type:
                                    getAudioMimeType(
                                        fakeFileName
                                    )
                            }
                        );


                    allAudioFiles.push(file);
                }

            }catch(error){

                console.error(
                    "ZIP error:",
                    error
                );

                alert(
                    "Could not read ZIP file: " +
                    zipFile.name
                );
            }
        }


        if(allAudioFiles.length === 0){

            hideLoading();

            alert(
                "No supported audio files found."
            );

            return;
        }


        clearExistingTracks();


        showLoading(
            "Creating " +
            allAudioFiles.length +
            " players..."
        );


        allAudioFiles.forEach(
            (file,index) => {

                createTrack(
                    file,
                    index
                );
            }
        );


        initializeTrackRouting();


        updateMaxDuration();

        checkPlayButtonState();


    }catch(error){

        console.error(error);

        alert(
            "Error loading folder."
        );

    }finally{

        hideLoading();
    }
}


/* =========================================================
   LOAD SELECTED FILES / ZIP
========================================================= */

async function loadSelectedFiles(fileList){

    showLoading(
        "Loading selected files..."
    );

    try{

        const normalAudio = [];

        const zipFiles = [];


        for(
            const file of Array.from(fileList)
        ){

            if(isAudioFile(file)){

                normalAudio.push(file);

            }else if(isZipFile(file)){

                zipFiles.push(file);
            }
        }


        const allAudioFiles = [
            ...normalAudio
        ];


        /*
            READ ZIP FILES
        */

        for(
            const zipFile of zipFiles
        ){

            showLoading(
                "Extracting " +
                zipFile.name +
                "..."
            );


            try{

                const zip =
                    await JSZip.loadAsync(zipFile);


                const entries =
                    Object.values(zip.files);


                for(
                    const entry of entries
                ){

                    if(entry.dir){
                        continue;
                    }


                    const fakeFileName =
                        entry.name
                            .split("/")
                            .pop();


                    if(!isAudioFile({
                        name: fakeFileName
                    })){
                        continue;
                    }


                    const blob =
                        await entry.async("blob");


                    const file =
                        new File(
                            [blob],
                            fakeFileName,
                            {
                                type:
                                    getAudioMimeType(
                                        fakeFileName
                                    )
                            }
                        );


                    allAudioFiles.push(file);
                }

            }catch(error){

                console.error(
                    "ZIP error:",
                    error
                );

                alert(
                    "Could not read ZIP: " +
                    zipFile.name
                );
            }
        }


        if(allAudioFiles.length === 0){

            hideLoading();

            alert(
                "No supported audio files selected."
            );

            return;
        }


        clearExistingTracks();


        showLoading(
            "Creating " +
            allAudioFiles.length +
            " players..."
        );


        allAudioFiles.forEach(
            (file,index) => {

                createTrack(
                    file,
                    index
                );
            }
        );


        initializeTrackRouting();


        updateMaxDuration();

        checkPlayButtonState();


    }catch(error){

        console.error(error);

        alert(
            "Error loading files."
        );

    }finally{

        hideLoading();
    }
}


/* =========================================================
   MIME TYPE
========================================================= */

function getAudioMimeType(fileName){

    const ext =
        fileName
            .toLowerCase()
            .split(".")
            .pop();


    const map = {

        mp3:"audio/mpeg",

        wav:"audio/wav",

        ogg:"audio/ogg",

        flac:"audio/flac",

        aac:"audio/aac",

        m4a:"audio/mp4",

        webm:"audio/webm",

        opus:"audio/ogg"
    };


    return map[ext] || "audio/*";
}


/* =========================================================
   MAX DURATION
========================================================= */

function updateMaxDuration(){

    let maxDuration = 0;


    tracks.forEach(
        track => {

            if(
                track.isLoaded &&
                isFinite(track.audio.duration)
            ){

                maxDuration =
                    Math.max(
                        maxDuration,
                        track.audio.duration
                    );
            }
        }
    );


    const seekbar =
        document.getElementById(
            "masterSeekbar"
        );


    seekbar.max = maxDuration;

    seekbar.disabled =
        tracks.length === 0 ||
        maxDuration <= 0;


    document.getElementById(
        "totalTime"
    ).textContent =
        formatTime(maxDuration);
}


/* =========================================================
   PLAY BUTTON STATE
========================================================= */

function checkPlayButtonState(){

    const button =
        document.getElementById(
            "playAllBtn"
        );


    const hasLoaded =
        tracks.some(
            track => track.isLoaded
        );


    button.disabled =
        !hasLoaded;
}


/* =========================================================
   PLAY ALL
========================================================= */

async function playAll(){

    if(tracks.length === 0){
        return;
    }


    const ctx =
        getAudioContext();


    if(!ctx){
        return;
    }


    try{

        if(ctx.state === "suspended"){
            await ctx.resume();
        }

    }catch(error){

        console.error(error);
    }


    /*
        Synchronize all tracks
    */

    const startTime =
        Number(
            document.getElementById(
                "masterSeekbar"
            ).value
        );


    tracks.forEach(
        track => {

            if(!track.isLoaded){
                return;
            }


            if(
                isFinite(startTime) &&
                startTime >= 0 &&
                startTime < track.audio.duration
            ){

                try{

                    track.audio.currentTime =
                        startTime;

                }catch(e){}
            }
        }
    );


    /*
        PLAY EVERY TRACK TOGETHER
    */

    const promises = [];


    tracks.forEach(
        track => {

            if(!track.isLoaded){
                return;
            }


            /*
                Apply current L/R state
                before playing.
            */

            updateChannelRouting(
                tracks.indexOf(track)
            );


            const promise =
                track.audio.play();


            if(promise){
                promises.push(
                    promise.catch(
                        error => {
                            console.warn(
                                "Play error:",
                                error
                            );
                        }
                    )
                );
            }
        }
    );


    await Promise.all(promises);


    isPlayingAll = true;
}


/* =========================================================
   PAUSE ALL
========================================================= */

function pauseAll(){

    tracks.forEach(
        track => {

            try{
                track.audio.pause();
            }catch(e){}
        }
    );


    isPlayingAll = false;
}


/* =========================================================
   CHECK ALL ENDED
========================================================= */

function checkAllEnded(){

    if(tracks.length === 0){
        return;
    }


    const anyPlaying =
        tracks.some(
            track =>
                !track.audio.paused &&
                !track.audio.ended
        );


    if(!anyPlaying){

        isPlayingAll = false;
    }
}


/* =========================================================
   MASTER SEEK
========================================================= */

document
    .getElementById("masterSeekbar")
    .addEventListener(
        "input",
        function(){

            if(masterUpdating){
                return;
            }


            const time =
                Number(this.value);


            tracks.forEach(
                track => {

                    if(
                        track.isLoaded &&
                        isFinite(track.audio.duration)
                    ){

                        try{

                            track.audio.currentTime =
                                Math.min(
                                    time,
                                    track.audio.duration
                                );

                        }catch(e){}
                    }
                }
            );


            document.getElementById(
                "currentTime"
            ).textContent =
                formatTime(time);
        }
    );


/* =========================================================
   MASTER PLAY
========================================================= */

document
    .getElementById("playAllBtn")
    .addEventListener(
        "click",
        playAll
    );


/* =========================================================
   MASTER PAUSE
========================================================= */

document
    .getElementById("pauseAllBtn")
    .addEventListener(
        "click",
        pauseAll
    );


/* =========================================================
   FOLDER INPUT
========================================================= */

document
    .getElementById("folderInput")
    .addEventListener(
        "change",
        async function(){

            if(this.files.length > 0){

                await loadFolderFiles(
                    this.files
                );
            }

            this.value = "";
        }
    );


/* =========================================================
   FILE INPUT
========================================================= */

document
    .getElementById("fileInput")
    .addEventListener(
        "change",
        async function(){

            if(this.files.length > 0){

                await loadSelectedFiles(
                    this.files
                );
            }

            this.value = "";
        }
    );


/* =========================================================
   RESET
========================================================= */

document
    .getElementById("resetBtn")
    .addEventListener(
        "click",
        () => {

            clearExistingTracks();
        }
    );


/* =========================================================
   MASTER TIME UPDATE
========================================================= */

setInterval(
    () => {

        if(tracks.length === 0){
            return;
        }


        const firstTrack =
            tracks.find(
                track => track.isLoaded
            );


        if(!firstTrack){
            return;
        }


        if(
            !firstTrack.audio.paused &&
            !firstTrack.audio.ended
        ){

            const current =
                firstTrack.audio.currentTime;


            masterUpdating = true;


            const seekbar =
                document.getElementById(
                    "masterSeekbar"
                );


            seekbar.value =
                Math.min(
                    current,
                    Number(seekbar.max) || current
                );


            document.getElementById(
                "currentTime"
            ).textContent =
                formatTime(current);


            masterUpdating = false;
        }


        checkAllEnded();

    },
    100
);

</script>

</body>
</html>
```

এই version-এ **Left/Right mix হবে না**—প্রতিটি track-এর original stereo channel আলাদা করে route করা হয়েছে। `Left` বা `Right` বন্ধ করলে সেই channel-এর output gain সরাসরি `0` হয়ে যায়।