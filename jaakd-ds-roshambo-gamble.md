---
layout: index
title: "Rock Paper Scissors: Gamble Mode"
heading: "Rock Paper Roshambo in JavaScript"
subheading: "Gamble Mode"
description: "Roshambo on Gamble Mode"
user-story: "As a player, I want to start with a balance and wager chips on each round so that I can gamble on the outcome of the game."
---




Which one will it be?
<a href="#" onclick="playRoshambo('rock')">rock</a>
<a href="#" onclick="playRoshambo('paper')">paper</a>
<a href="#" onclick="playRoshambo('scissors')">scissors</a>

<br/>
Wager: <input type="number" id="wager" value="10" min="1">

<div id="balance"></div>
<div id="results"></div>
<div id="history"></div>
<br/>

<script>
games = JSON.parse(localStorage.getItem('games')) || [];
    balance = parseInt(localStorage.getItem('balance')) || 100;
    serverGesture = 'scissors';
    document.getElementById('balance').innerHTML = 'Balance: ' + balance;
    playRoshambo = function(clientGesture){
        wager = parseInt(document.getElementById('wager').value) || 0;
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
            balance += wager;
        }
        if (result=='loss') {
            balance -= wager;
        }
        localStorage.setItem('balance', balance);
        document.getElementById('balance').innerHTML = 'Balance: ' + balance;
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