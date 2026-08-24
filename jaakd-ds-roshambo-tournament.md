---
layout: index
title: "Rock Paper Scissors: Tournament Bracket Mode"
heading: "Rock Paper Roshambo in JavaScript"
subheading: "Tournament Bracket Mode"
description: "Roshambo on Tournament Bracket Mode"
user-story: "As a player, I want to advance through a bracket of rounds by winning each match so that I can be crowned tournament champion."
---




Which one will it be?
<a href="#" onclick="playRoshambo('rock')">rock</a>
<a href="#" onclick="playRoshambo('paper')">paper</a>
<a href="#" onclick="playRoshambo('scissors')">scissors</a>

<div id="bracket"></div>
<div id="results"></div>
<div id="history"></div>
<br/>

<script>
games = JSON.parse(localStorage.getItem('games')) || [];
    TOTAL_ROUNDS = 3;
    round = parseInt(localStorage.getItem('round')) || 1;
    serverGesture = 'scissors';
    document.getElementById('bracket').innerHTML = 'Round ' + round + ' of ' + TOTAL_ROUNDS;
    playRoshambo = function(clientGesture){
        if (clientGesture=='rock') {
            result = "win";
        }
        if (clientGesture=='paper') {
            result = "loss";
        }
        if (clientGesture=='scissors') {
            result = "tie";
        }
        if (result=='win') {
            round++;
            if (round > TOTAL_ROUNDS) {
                document.getElementById('bracket').innerHTML = 'Tournament Champion!';
                round = 1;
            } else {
                document.getElementById('bracket').innerHTML = 'Round ' + round + ' of ' + TOTAL_ROUNDS;
            }
        }
        if (result=='loss') {
            round = 1;
            document.getElementById('bracket').innerHTML = 'Eliminated! Round ' + round + ' of ' + TOTAL_ROUNDS;
        }
        localStorage.setItem('round', round);
        document.getElementById('results').innerHTML = result;
        saveGame(clientGesture, serverGesture, result);
    }

    saveGame = function(clientGesture, serverGesture, result) {
        game = {
            clientGesture: clientGesture,
            serverGesture: serverGesture,
            result: result,
            time: new Date()
        };
        games.push(game);
        localStorage.setItem('games', JSON.stringify(games));
        showHistory();
    }

    deleteGame = function(time) {
        games = games.filter(game => game.time != time);
        localStorage.setItem('games', JSON.stringify(games));
        showHistory();
    }

    showHistory = function() {
        historyText = "";
        for (game of games) {
            historyText += "<div>";
            historyText += game.clientGesture + " | ";
            historyText += game.serverGesture + " | ";
            historyText += game.result + " | ";
            historyText += game.time + " | ";
            historyText += "<a href='#' onclick=\"deleteGame('" + game.time + "')\">delete</a>";
            historyText += "</div>";
        }
    document.getElementById('history').innerHTML = historyText;
    }

    showHistory();
</script>