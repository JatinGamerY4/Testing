<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Voting System</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin-top: 50px;
      background-color: #f0f0f0;
    }
    h1 {
      margin-bottom: 20px;
    }
    button {
      font-size: 18px;
      padding: 10px 20px;
      margin: 10px;
      cursor: pointer;
    }
    #result {
      margin-top: 30px;
      font-size: 22px;
      font-weight: bold;
      color: green;
    }
  </style>
</head>
<body>

  <h1>Vote for Your Favorite Option</h1>
  <button onclick="vote('option1')" id="btn1">Option 1</button>
  <button onclick="vote('option2')" id="btn2">Option 2</button>

  <div id="result"></div>

  <script>
    let votes1 = localStorage.getItem('votes1') || 0;
    let votes2 = localStorage.getItem('votes2') || 0;
    const hasVoted = localStorage.getItem('hasVoted');

    function updateResult() {
      const resultDiv = document.getElementById('result');
      if (votes1 > votes2) {
        resultDiv.textContent = "Option 1 is winning!";
      } else if (votes2 > votes1) {
        resultDiv.textContent = "Option 2 is winning!";
      } else {
        resultDiv.textContent = "It's a tie!";
      }
    }

    function vote(option) {
      if (hasVoted) {
        alert("You have already voted!");
        return;
      }

      if (option === 'option1') {
        votes1++;
        localStorage.setItem('votes1', votes1);
      } else if (option === 'option2') {
        votes2++;
        localStorage.setItem('votes2', votes2);
      }

      localStorage.setItem('hasVoted', true);
      document.getElementById("btn1").disabled = true;
      document.getElementById("btn2").disabled = true;
      updateResult();
    }

    // Disable buttons if already voted
    if (hasVoted) {
      document.getElementById("btn1").disabled = true;
      document.getElementById("btn2").disabled = true;
    }

    updateResult();
  </script>

</body>
</html>
