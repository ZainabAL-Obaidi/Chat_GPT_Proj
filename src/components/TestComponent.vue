<template>
  <div>
    <h1>English Level Test</h1>
    <div v-if="currentQuestion">
      <p>{{ currentQuestion }}</p>
      <input v-model="userAnswer" @keyup.enter="submitAnswer" />
      <button @click="submitAnswer">Submit</button>
    </div>
    <div v-else>
      <p>{{ feedback }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentQuestion: null,
      userAnswer: '',
      feedback: '',
      questionIndex: 0,
      answers: [],
    };
  },
  methods: {
    async fetchQuestion() {
      try {
        const response = await fetch('/api/get-question', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ questionIndex: this.questionIndex }),
        });
        const data = await response.json();
        this.currentQuestion = data.question;
      } catch (error) {
        console.error('Error fetching question:', error);
      }
    },
    async submitAnswer() {
      this.answers.push(this.userAnswer);
      this.userAnswer = '';
      this.questionIndex++;
      if (this.questionIndex < 3) {
        await this.fetchQuestion();
      } else {
        await this.getFeedback();
      }
    },
    async getFeedback() {
      try {
        const response = await fetch('/api/get-feedback', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ answers: this.answers }),
        });
        const data = await response.json();
        this.feedback = data.feedback;
        this.currentQuestion = null;
      } catch (error) {
        console.error('Error fetching feedback:', error);
      }
    },
  },
  async created() {
    await this.fetchQuestion();
  },
};
</script>
