<script>
import {ref, onMounted} from 'vue';
import jsQR from 'jsqr';
import {f7} from 'framework7-vue';

export default {
  setup() {
    const camera = ref(null);
    const canvas = ref(null);
    const output = ref("");

    const openAlert = (text) => {
      f7.dialog.alert(text);
    };
    const readNFC = async () => {
      if ('NDEFReader' in window) {
        openAlert("Web NFC is supported.")
        output.value = "Web NFC is supported.";
        try {
          const ndef = new NDEFReader();
          output.value = "NDEFReader.";
          await ndef.scan();
          output.value = "Scan started, touch an NFC tag to read data.";
          ndef.addEventListener("readingerror", () => {
            output.value = "Argh! Cannot read data from the NFC tag. Try another one?";
          });

          ndef.addEventListener("reading", ({message, serialNumber}) => {
            output.value = `> Serial Number: ${serialNumber} `;
            // output.value = output.value + `> Records: (${message.records.length}) <br>`;
          });
        } catch (error) {
          output.value =  "Argh! " + error ;
        }
      } else {
        output.value = "Web NFC is not supported.";
      }
    };

    const startScanning = () => {
      const video = camera.value;
      const canvasElement = canvas.value;
      const canvasContext = canvasElement.getContext('2d');

      navigator.mediaDevices.getUserMedia({video: {facingMode: "environment"}})
          .then(stream => {
            video.srcObject = stream;
            video.play();
            scanQRCode();
          })
          .catch(error => {
            console.error('Error accessing the camera', error);
            // Use Framework7's dialog component for user feedback
            openAlert('Error accessing the camera' + error);
          });

      const scanQRCode = () => {
        const tick = () => {
          if (video.readyState === video.HAVE_ENOUGH_DATA) {
            canvasElement.hidden = false;
            canvasElement.height = video.videoHeight;
            canvasElement.width = video.videoWidth;
            canvasContext.drawImage(video, 0, 0, canvasElement.width, canvasElement.height);
            const imageData = canvasContext.getImageData(0, 0, canvasElement.width, canvasElement.height);
            const code = jsQR(imageData.data, imageData.width, imageData.height, {
              inversionAttempts: "dontInvert",
            });

            if (code) {
              console.log('QR Code detected:', code.data);
              openAlert('QR Code: ' + code.data);
              video.srcObject.getTracks().forEach(track => track.stop());
            } else {
              requestAnimationFrame(tick);
            }
          } else {
            requestAnimationFrame(tick);
          }
        };
        requestAnimationFrame(tick);
      };
    };


    return {camera, canvas, startScanning, readNFC, output};
  }
}
</script>
<template>
  <f7-page name="home">
    <!-- Top Navbar -->
    <f7-navbar large :sliding="false">
      <f7-nav-left>
        <f7-link icon-ios="f7:menu" icon-md="material:menu" panel-open="left"></f7-link>
      </f7-nav-left>
      <f7-nav-title sliding>Hello QRCode Scanner</f7-nav-title>
      <f7-nav-right>
        <f7-link icon-ios="f7:menu" icon-md="material:menu" panel-open="right"></f7-link>
      </f7-nav-right>
      <f7-nav-title-large>Hello QRCode Scanner</f7-nav-title-large>
    </f7-navbar>

    <!-- Page content-->
    <f7-navbar title="Scan QR Code"></f7-navbar>
    <f7-block>
      <video ref="camera" hidden></video>
      <canvas ref="canvas" hidden></canvas>
      <f7-button @click="startScanning" fill>Start Scanning</f7-button>
      <f7-button @click="readNFC" large>Read NFC Tag</f7-button>
    </f7-block>

    <f7-block>
      <p>NFC Output: {{ output }}</p>
    </f7-block>
    <f7-block-title>Navigation</f7-block-title>
    <f7-list strong inset dividersIos>
      <f7-list-item link="/about/" title="About"></f7-list-item>
      <f7-list-item link="/form/" title="Form"></f7-list-item>
    </f7-list>

    <f7-block-title>拍照</f7-block-title>
    <f7-list>
      <f7-list-item>
        <f7-input type="file" accept="image/*" capture="camera" id="cameraInput"></f7-input>
      </f7-list-item>
    </f7-list>

    <f7-block-title>Modals</f7-block-title>
    <f7-block class="grid grid-cols-2 grid-gap">
      <f7-button fill popup-open="#my-popup">Popup</f7-button>
      <f7-button fill login-screen-open="#my-login-screen">Login Screen</f7-button>
    </f7-block>

    <f7-block-title>Panels</f7-block-title>
    <f7-block class="grid grid-cols-2 grid-gap">
      <f7-button fill panel-open="left">Left Panel</f7-button>
      <f7-button fill panel-open="right">Right Panel</f7-button>
    </f7-block>

    <f7-list strong inset dividersIos>
      <f7-list-item
          title="Dynamic (Component) Route"
          link="/dynamic-route/blog/45/post/125/?foo=bar#about"
      ></f7-list-item>
      <f7-list-item
          title="Default Route (404)"
          link="/load-something-that-doesnt-exist/"
      ></f7-list-item>
      <f7-list-item
          title="Request Data & Load"
          link="/request-and-load/user/123456/"
      ></f7-list-item>
    </f7-list>
  </f7-page>
</template>