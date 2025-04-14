import { useRef, useEffect, useState } from "react";
import "./App.css";

function App() {
  const videoRef = useRef(null);
  const [videoURL, setVideoURL] = useState(null);
  const [showSpeed, setShowSpeed] = useState(false);
  const [overlayText, setOverlayText] = useState("");
  const [saturation, setSaturation] = useState(100); // 100% by default


  const handleFileChange = (e) => {
    const file = e.target.files[0];
    if (file) {
      const url = URL.createObjectURL(file);
      setVideoURL(url);
    }
  };

  const showOverlay = (text) => {
    setOverlayText(text);
    setShowSpeed(true);
    setTimeout(() => {
      setShowSpeed(false);
    }, 2000);
  };

  const toggleFullScreen = () => {
    const video = videoRef.current;
    if (!document.fullscreenElement) {
      video.requestFullscreen().catch((err) => {
        console.error("Failed to enter fullscreen:", err);
      });
    } else {
      document.exitFullscreen();
    }
  };

  const togglePlayPause = () => {
    const video = videoRef.current;
    if (video.paused) {
      video.play();
    } else {
      video.pause();
    }
  };

  useEffect(() => {
    const handleKeyDown = (e) => {
      const video = videoRef.current;
      const tag = document.activeElement.tagName;
  
      if (!video || tag === "INPUT" || tag === "TEXTAREA") return;
  
      // const keysToPrevent = [
      //   " ", "Escape", "f", "F", "ArrowUp", "ArrowDown",
      //   "ArrowLeft", "ArrowRight", "Enter"
      // ];
  
      if (e.key === " " && tag !== "INPUT" && tag !== "TEXTAREA") {
        e.preventDefault(); // prevent page scroll
      }
      
  
      // Spacebar Play/Pause
      if (e.key === " " && !e.repeat) {
        togglePlayPause();
      }

      // if (e.key === 'ArrowRight') {
      //   videoRef.current.currentTime += 10;
      // }
  
      // // ArrowLeft: Skip backward 10 seconds
      // if (e.key === 'ArrowLeft') {
      //   videoRef.current.currentTime -= 10;
      // }
  
    
  
      // Arrow key skipping
      switch (e.key) {
        case "ArrowRight":
          video.currentTime += 5 ;
          showOverlay(">> 5s");
          break;
        case "ArrowLeft":
          video.currentTime -= 5;
          showOverlay("<< 5s");
          break;
        case "+":
          if (video.playbackRate < 3) {
            video.playbackRate += 0.25;
            showOverlay(`${video.playbackRate.toFixed(2)}x`);
          }
          break;
        case "-":
          if (video.playbackRate > 0.25) {
            video.playbackRate -= 0.25;
            showOverlay(`${video.playbackRate.toFixed(2)}x`);
          }
          break;
        case "ArrowUp":
          video.volume = Math.min(1, video.volume + 0.1);
          showOverlay(`🔊 ${Math.round(video.volume * 100)}%`);
          break;
        case "ArrowDown":
          video.volume = Math.max(0, video.volume - 0.1);
          showOverlay(`🔉 ${Math.round(video.volume * 100)}%`);
          break;
        case "Enter":
          toggleFullScreen();
          break;
      }
    };
  
    window.addEventListener("keydown", handleKeyDown);
    return () => window.removeEventListener("keydown", handleKeyDown);
  }, []);
  

  return (
    <div className="container">
      <h1>🎬 Ashish Video Player</h1>
      <label className="custom-file-upload">
  🎥 Choose Video File
  <input type="file" accept="video/*" onChange={handleFileChange} />
</label>

      {videoURL ? (
        <div className="video-wrapper">
        <video
  ref={videoRef}
  className="video-player"
  src={videoURL}
  style={{ filter: `saturate(${saturation}%)` }}
  controls
  tabIndex={-1}
  onClick={(e) => {
    e.preventDefault();
    e.stopPropagation();
    videoRef.current.blur(); // prevent focus stealing
  }}
  onDoubleClick={(e) => {
    e.preventDefault();
    togglePlayPause(); // Double-click toggles play/pause
  }}
/>
<div className="slider-container">
  <label htmlFor="saturation">🎨 Saturation: {saturation}%</label>
  <input
    type="range"
    id="saturation"
    min="0"
    max="200"
    value={saturation}
    onChange={(e) => setSaturation(e.target.value)}
  />
</div>


          {showSpeed && <div className="overlay">{overlayText}</div>}
        </div>
      ) : (
        <p className="placeholder">Choose a video to play</p>
      )}

      <div className="info">
        <p>
          <strong>Keyboard Shortcuts:</strong>
        </p>
        <ul>
          <li>Spacebar: Play / Pause</li>
          <li>&gt; / &lt; : Skip +5s / -5s</li>
          <li>+ / - : Speed up / down</li>
          <li>↑ / ↓ : Volume up / down</li>
          <li>Enter: fullscreen / esc</li>
          <li>Double-click: Play / Pause</li>
        </ul>
      </div>
    </div>
  );
}

export default App;
