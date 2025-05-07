<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Happy Birthday!</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background: url('https://www.example.com/path/to/your/background-image.jpg') no-repeat center center fixed;
      background-size: cover;
      overflow-x: hidden;
      padding: 20px;
      transition: background-color 2s ease-in-out;
    }
    #inputForm, #cake {
      margin-top: 50px;
    }
    .balloon {
      display: inline-block;
      width: 60px;
      height: 80px;
      margin: 10px;
      background: radial-gradient(circle at 30% 30%, #ff5f5f, #d60000);
      border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
      position: relative;
      cursor: pointer;
      box-shadow: inset -5px -5px 15px rgba(255,255,255,0.6),
                  0 8px 15px rgba(0, 0, 0, 0.2);
      animation: float 3s ease-in-out infinite;
      transition: transform 0.2s ease;
    }
    .balloon:hover {
      transform: scale(1.05);
    }
    .balloon::after {
      content: '';
      position: absolute;
      bottom: -20px;
      left: 50%;
      transform: translateX(-50%);
      width: 4px;
      height: 20px;
      background: #333;
    }
    @keyframes float {
      0%, 100% { transform: translateY(0); }
      50% { transform: translateY(-20px); }
    }
    @keyframes pop {
      0% { opacity: 1; transform: scale(1); }
      100% { opacity: 0; transform: scale(1.5); }
    }
    .popped {
      animation: pop 0.4s forwards;
    }
    .hidden {
      display: none;
    }
    #cake {
      margin-top: 30px;
    }
    .candle {
      width: 5px;
      height: 20px;
      background: orange;
      margin: 2px;
      display: inline-block;
    }
    #confetti {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
      z-index: 1000;
    }
    .confetti-piece {
      position: absolute;
      width: 10px;
      height: 10px;
      background: #ffcc00;
      animation: confetti-fall 2s linear infinite;
      opacity: 0;
    }
    @keyframes confetti-fall {
      0% { transform: translateY(0); opacity: 1; }
      100% { transform: translateY(100vh); opacity: 0; }
    }
    #profileImage {
      margin-top: 20px;
      width: 150px;
      height: 150px;
      border-radius: 50%;
      object-fit: cover;
      border: 5px solid #fff;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    }
  </style>
</head>
<body>

<h1>🎈 Birthday Balloon Pop Game 🎈</h1>

<div id="inputForm">
  <label>Who are you wishing?</label><br>
  <input type="text" id="nameInput" placeholder="Recipient's Name"><br><br>
  <label>How old are they turning?</label><br>
  <input type="number" id="ageInput" min="1" max="120"><br><br>
  
  <!-- Image upload input -->
  <label>Upload their picture:</label><br>
  <input type="file" id="imageInput" accept="image/*"><br><br>
  
  <button id="startBtn">Start!</button>
</div>

<div id="message" class="hidden"></div>
<div id="balloons" class="hidden"></div>

<div id="cake" class="hidden">
  <h2>🎂 Happy Birthday! 🎂</h2>
  <div id="candles"></div>
</div>

<!-- Happy Birthday song as background music -->
<audio id="backgroundMusic" src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" loop></audio>
<audio id="popSound" src="https://www.fesliyanstudios.com/play-mp3/387" preload="auto"></audio>
<audio id="happyBirthdaySong" src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" preload="auto"></audio>

<!-- Button for generating the shareable link -->
<button id="shareBtn" class="hidden">Share the link with the birthday person!</button>

<div id="confetti" class="hidden"></div>

<script>
  document.getElementById("startBtn").addEventListener("click", startBirthday);

  function startBirthday() {
    const name = document.getElementById("nameInput").value.trim();
    const age = parseInt(document.getElementById("ageInput").value);
    const imageInput = document.getElementById("imageInput").files[0];

    if (!name || isNaN(age) || age < 1 || age > 120) {
      alert("Please enter a valid name and age between 1 and 120.");
      return;
    }

    document.getElementById("inputForm").classList.add("hidden");
    document.getElementById("message").classList.remove("hidden");
    document.getElementById("balloons").classList.remove("hidden");
    document.getElementById("shareBtn").classList.remove("hidden"); // Show the share button

    // Display the uploaded image (if available)
    if (imageInput) {
      const imageUrl = URL.createObjectURL(imageInput);
      const imageElement = document.createElement("img");
      imageElement.id = "profileImage";
      imageElement.src = imageUrl;
      document.getElementById("message").appendChild(imageElement);
    }

    document.getElementById("message").innerHTML += `<h2>🎉 Happy ${age}th Birthday, ${name}! 🎉`;

    // Start background music when the game begins
    document.getElementById("backgroundMusic").play();
    document.getElementById("happyBirthdaySong").play();

    const balloonsContainer = document.getElementById("balloons");
    balloonsContainer.innerHTML = "";

    for (let i = 0; i < age; i++) {
      const balloon = document.createElement("div");
      balloon.className = "balloon";
      balloon.dataset.index = i;  // Add an index to the balloon for confetti position

      balloon.addEventListener("click", function () {
        if (!balloon.classList.contains("popped")) {
          // Play the balloon popping sound
          playPopSound();
          balloon.classList.add("popped");

          // Create confetti from this balloon position
          createConfettiFromBalloon(balloon);

          // After the pop animation ends, remove the balloon immediately
          balloon.addEventListener("animationend", () => {
            balloon.remove();
            checkAllPopped();
          });
        }
      });

      balloonsContainer.appendChild(balloon);
    }

    // Start the background color change every 5 seconds
    startColorChange();
  }

  function checkAllPopped() {
    const remaining = document.querySelectorAll(".balloon").length;
    if (remaining === 0) {
      showCake();
      playHappyBirthdaySong();
    }
  }

  function showCake() {
    const age = parseInt(document.getElementById("ageInput").value);
    const cake = document.getElementById("cake");
    const candles = document.getElementById("candles");

    cake.classList.remove("hidden");
    candles.innerHTML = "";

    for (let i = 0; i < age; i++) {
      const candle = document.createElement("div");
      candle.className = "candle";
      candles.appendChild(candle);
    }
  }

  function playPopSound() {
    const sound = document.getElementById("popSound");
    sound.currentTime = 0;
    sound.play();
  }

  function playHappyBirthdaySong() {
    const song = document.getElementById("happyBirthdaySong");
    song.currentTime = 0;
    song.play();
  }

  function createConfettiFromBalloon(balloon) {
    const confettiContainer = document.getElementById("confetti");
    confettiContainer.classList.remove("hidden");

    // Generate confetti pieces at the location of the popped balloon
    const balloonRect = balloon.getBoundingClientRect();
    for (let i = 0; i < 10; i++) {
      const confettiPiece = document.createElement("div");
      confettiPiece.className = "confetti-piece";
      confettiPiece.style.left = (balloonRect.left + Math.random() * balloonRect.width) + "px";
      confettiPiece.style.top = (balloonRect.top + Math.random() * balloonRect.height) + "px";
      confettiPiece.style.animationDuration = Math.random() * 3 + 2 + "s";
      confettiPiece.style.animationDelay = Math.random() * 0.5 + "s"; // Adds slight delay for each confetti
      confettiContainer.appendChild(confettiPiece);
    }
  }

  function startColorChange() {
    let colorIndex = 0;
    const colors = ["#FF7F50", "#FFD700", "#ADFF2F", "#32CD32", "#8A2BE2"];
    
    setInterval(() => {
      // Fade out
      document.body.style.transition = "background-color 2s ease-in-out";
      document.body.style.backgroundColor = colors[colorIndex];

      // Fade in next color after transition
      colorIndex = (colorIndex + 1) % colors.length;
    }, 5000); // Change color every 5 seconds
  }

  function downloadPage() {
    const element = document.createElement('a');
    const content = document.documentElement.outerHTML;
    const file = new Blob([content], { type: 'text/html' });
    element.href = URL.createObjectURL(file);
    element.download = "birthday_greeting.html";
    element.click();
  }

  // Shareable link creation
  document.getElementById("shareBtn").addEventListener("click", () => {
    const name = document.getElementById("nameInput").value.trim();
    const age = parseInt(document.getElementById("ageInput").value);

    if (name && !isNaN(age)) {
      const link = `https://yourdomain.com/birthday?name=${encodeURIComponent(name)}&age=${age}`;
      prompt("Share this link with the birthday person:", link);
    }
  });
</script>

</body>
</html>
