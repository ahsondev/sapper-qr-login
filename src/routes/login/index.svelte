<script>
  import QRScanner from '../../components/QRreader.svelte';
  import Sockio from "socket.io-client";

  function handle_qr_read(qr_event) {
    // holdup_flag.set(true);
    console.log(qr_event);
    try {
      let VALUE = qr_event.detail.code;
      // console.log(VALUE);
      switch (VALUE) {
        case "START":
          dispatch("start");
          break;
        case "STOP":
          dispatch("stop");
          break;
        default:
          handle_non_command_qr(VALUE);
          break;
      }
    } catch (error) {
      console.log(error);
    }
  }

  const socket = Sockio("localhost:3003");
  socket.on("connect", function handle_login(Data) {
			console.log("data from server:");
		});

</script>

<div>
  <h1>Login View</h1>
  <input type='text'/>
  <QRScanner
      on:qr_read={handle_qr_read}
      video_height="300"
      video_width="350"
      facing_mode="user"
    >
      <div class="placeholder">No cameras loaded!</div>
    </QRScanner>
</div>

<style></style>