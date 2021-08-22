<script>
    import jsQR from "jsqr";
    import { onMount } from "svelte";
    import { onDestroy } from "svelte";
    
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();

    onDestroy(()=>
    {
        // console.log("DESTROUING QR SCANNER");
        stop_scanning = true;
            // console.log("Stopped scanning");
            try {  
            _streamCopy.getVideoTracks()[0].stop();
            } catch (error) {
                
            }
    });
    
    let stop_scanning;
    export let video_height;
    export let video_width;
    export let facing_mode;

    function handle_qr_read(val)
    {
        dispatch('qr_read', {code:val});
    }


    var _streamCopy = null;

    let QRNode;
    let video =null;
    //////////

    onMount(() => 
    {
        // console.log("MOUNTING QR READER");
        video = document.createElement("video");
                video.setAttribute("playsinline", true); // required to tell iOS safari we don't want fullscreen
                video.setAttribute('height', video_height+'px');
                video.setAttribute('width', video_width+'px');
        var canvasElement = document.getElementById("canvas");
        var canvas = canvasElement.getContext("2d");
        var loadingMessage = document.getElementById("loadingMessage");
        var outputContainer = document.getElementById("output");
        var outputMessage = document.getElementById("outputMessage");
        var outputData = document.getElementById("outputData");
       
        // Use facingMode: environment to attemt to get the front camera on phones
        navigator.mediaDevices
            .getUserMedia({ video: { facingMode: facing_mode } })
            .then(function (stream) {
                video.srcObject = stream;
                _streamCopy = stream;
                video.setAttribute("playsinline", true); // required to tell iOS safari we don't want fullscreen
                video.setAttribute('height', video_height+'px');
                video.setAttribute('width', video_width+'px');
                
                video.play();
                if (!stop_scanning) {
                    
                    requestAnimationFrame(tick);
                }else
                {
                    console.log("SCANNING STOPPED")
                }
            });

        function drawLine(begin, end, color) {
            canvas.beginPath();
            canvas.moveTo(begin.x, begin.y);
            canvas.lineTo(end.x, end.y);
            canvas.lineWidth = 4;
            canvas.strokeStyle = color;
            canvas.stroke();
        }

        function tick() {
            loadingMessage.innerText = "⌛ Loading video...";
            if (video.readyState === video.HAVE_ENOUGH_DATA) {
                loadingMessage.hidden = true;
                canvasElement.hidden = false;
                outputContainer.hidden = false;

                canvasElement.height = video_height;
                canvasElement.width = video_width;
                canvas.drawImage(
                    video,
                    0,
                    0,
                    canvasElement.width,
                    canvasElement.height
                );
                var imageData = canvas.getImageData(
                    0,
                    0,
                    canvasElement.width,
                    canvasElement.height
                );
                var code = jsQR(
                    imageData.data,
                    imageData.width,
                    imageData.height,
                    {
                        inversionAttempts: "dontInvert",
                    }
                );
                if (code) {
                    send_qr_result(code);
                    drawLine(
                        code.location.topLeftCorner,
                        code.location.topRightCorner,
                        "#FF3B58"
                    );
                    drawLine(
                        code.location.topRightCorner,
                        code.location.bottomRightCorner,
                        "#FF3B58"
                    );
                    drawLine(
                        code.location.bottomRightCorner,
                        code.location.bottomLeftCorner,
                        "#FF3B58"
                    );
                    drawLine(
                        code.location.bottomLeftCorner,
                        code.location.topLeftCorner,
                        "#FF3B58"
                    );
                    outputMessage.hidden = true;
                    outputData.parentElement.hidden = false;
                    outputData.innerText = code.data;
                } else {
                    outputMessage.hidden = false;
                    outputData.parentElement.hidden = true;
                }
            }

            if (!stop_scanning) 
            {
                requestAnimationFrame(tick);
            } else
            {
                console.log("SCANNING STOPPED")
                console.log("STOPPING STREAM");
                _streamCopy.getVideoTracks()[0].stop();
                // video.remove();
            }
        }
    });

    function send_qr_result(Code) 
    {
        
        handle_qr_read(Code.data);
        console.log(Code.data)
    }

    export const close_scanner = function () {};

</script>
<style>
    #canvas{
        margin:8pt;
    }
    .scanner_container{
        display:flex;
        flex-direction: column;
        align-items: center;
    }
    
</style>
<div class = scanner_container bind:this={QRNode}>
    <div id="loadingMessage" />
    
    <canvas id="canvas" height={video_height} width={video_width} />
    <div class = "content">

    </div>
   
    <div id="output">
        <div id="outputMessage" />
        <div hidden>
            
            <span id="outputData" />
        </div>
    </div>
  
</div>

