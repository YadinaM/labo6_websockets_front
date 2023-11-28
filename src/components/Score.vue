<script setup>
    import { ref, onMounted } from 'vue';

    let selectedTeam = ref('Team A');
    let scores = ref({
        'Team A': 0,
        'Team B': 0
    });

    let scoreInput = ref(0);

    let socket = null;

    onMounted(() => {
        // Connect to the WebSocket server
        socket = new WebSocket('ws://localhost:3000/primus');

        socket.onmessage = (event) => {
            let data = JSON.parse(event.data);
            if (data.action === "updateScore") {
            scores.value[data.team] = data.score;
            }
        };
    });

    const adjustScore = (team, value) => {
        scores.value[team] = value;

        // Send updated score to the server
        sendData({ action: "updateScore", team, score: scores.value[team] });
        scoreInput.value = 0;

    };

    const sendData = (data) => {
        // Send the data to the server using WebSockets
        socket.send(JSON.stringify(data));
    };
</script>

<template>
   <div>
    <label for="teams">Select a Team:</label>
    <select v-model="selectedTeam" id="teams">
      <option value="Team A">Team A</option>
      <option value="Team B">Team B</option>
    </select>

    <div>
      <h2>{{ selectedTeam }} Score: {{ scores[selectedTeam] }}</h2>
      <input type="number" v-model="scoreInput" />
      <button @click="adjustScore(selectedTeam, scoreInput)">Update Score</button>
    </div>
  </div>
</template>

<style scoped>

</style>
