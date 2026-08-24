---
layout: index
title: "Rock Paper Scissors: Intro Mode"
heading: "Fun Rock Paper Scissors Game in JavaScript"
subheading: "Intro Level Game"
description: "Roshambo on Intro Mode"
user-story: "As a player, I want to choose rock, paper, or scissors, play against the computer, and see whether I win, lose, or tie so that I can immediately understand the outcome of each round."
---

<p>Which one will it be?</p>
<a href="#" onclick="playRoshambo('rock')">rock</a>
<a href="#" onclick="playRoshambo('scissors')">scissors</a>
<a href="#" onclick="playRoshambo('paper')">paper</a>
<br/>
<div id="results"></div>
<script>
    playRoshambo = function(clientGesture){
        if (clientGesture=='rock') {
            result = "tie";
        } // end if
        if (clientGesture=='scissors') {
            result = "lose";        
        } // end if
        if (clientGesture=='paper') {
            result = "win";        
        } // end if
        document.getElementById('results').innerHTML = result;
    } // end method
</script>
