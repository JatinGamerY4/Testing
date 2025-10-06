<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Redirecting…</title>

  <!-- Meta refresh fallback -->
  <meta http-equiv="refresh" content="0; url=./Poppy%20Playtime%20Chapter%204%20Main%20Menu.mp4">

  <!-- Mobile-friendly viewport -->
  <meta name="viewport" content="width=device-width,initial-scale=1">

  <style>
    body { font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; 
           display:flex;align-items:center;justify-content:center;height:100vh;margin:0;
           background:#111;color:#fff; }
    .box { text-align:center; max-width:700px; padding:20px; }
    a { color:#8fd3ff; text-decoration:underline; }
  </style>
</head>
<body>
  <div class="box">
    <h1>Redirecting to your video…</h1>
    <p>If the redirect doesn't happen automatically, <a id="fallback-link" href="./Poppy%20Playtime%20Chapter%204%20Main%20Menu.mp4">click here to open the video</a>.</p>
    <p style="font-size:0.9em;opacity:0.8">If you want the video to play in-browser, make sure GitHub Pages serves this file (or host the MP4 on a raw/hosting URL).</p>
  </div>

  <script>
    // JavaScript redirect (faster & reliable for browsers that allow JS)
    (function(){
      try {
        var target = './Poppy%20Playtime%20Chapter%204%20Main%20Menu.mp4';
        // Small delay to allow user to see message briefly
        setTimeout(function(){ window.location.replace(target); }, 100);
      } catch(e){
        // leave the fallback link visible
        console.error(e);
      }
    })();
  </script>

  <!-- For users with JS disabled: the meta refresh and the link above work -->
</body>
</html>
