<template>
  <div style="width:80%;margin-left:10%">
    <div class="shadow_box">
      <label>GPT API Key : </label>
      <input v-model="apiKey" />
    </div>
    <div style="margin-top:20px;margin-bottom: 20px;" class="shadow_box">
      <label>Titre d'article :</label>
      <input v-model="articleTitle" />
    </div>

    <button @click="callGPT">Générer</button>
    <div v-if="loading">
      <p class="loading">Chargement...</p>
    </div>
    <div v-html="markdownToHtml(articleContent)" style="white-space:pre-wrap; text-align:justify;" class="generate_text">
    </div>
  </div>
</template>

<script>
import showdown from 'showdown';
export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      apiKey: '',
      articleTitle: '',
      articleContent: '',
      prompt: `I Want You To Act As A Content Writer Very Proficient SEO Writer Writes Fluently French. First Create Two Tables. First Table Should be the Outline of the Article and the Second Should be the Article. Bold the Heading of the Second Table using Markdown language. Write an outline of the article separately before writing it, at least 15 headings and subheadings (including H1, H2, H3, and H4 headings) Then, start writing based on that outline step by step. Write a 2000-word 100% Unique, SEO-optimized, Human-Written article in French with at least 15 headings and subheadings (including H1, H2, H3, and H4 headings) that covers the topic provided in the Prompt. Write The article In Your Own Words Rather Than Copying And Pasting From Other Sources. Consider perplexity and burstiness when creating content, ensuring high levels of both without losing specificity or context. Use fully detailed paragraphs that engage the reader. Write In A Conversational Style As Written By A Human (Use An Informal Tone, Utilize Personal Pronouns, Keep It Simple, Engage The Reader, Use The Active Voice, Keep It Brief, Use Rhetorical Questions, and Incorporate Analogies And Metaphors). End with a conclusion paragraph and 5 unique FAQs After The Conclusion. this is important to Bold the Title and all headings of the article, and use appropriate headings for H tags.
Now Write An Article On This Topic : '`,
      loading: false
    }
  },
  methods: {
    async callGPT() {
      this.loading = true;
      this.articleContent = ""
      const response = await fetch('https://api.openai.com/v1/completions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': 'Bearer ' + this.apiKey,
        },
        body: JSON.stringify({
          model: "text-davinci-003",
          prompt: this.prompt + this.articleTitle + "'. Reply in French. ",
          max_tokens: 3750,
          stream: true
        })
      });
      const reader = response.body?.pipeThrough(new TextDecoderStream()).getReader();
        if (!reader) return;
        while (true) {
          const { value, done } = await reader.read();
          if (done) break;
          let dataDone = false;
          const arr = value.split('\n');
          arr.forEach((data) => {
            if (data.length === 0) return;
            if (data.startsWith(':')) return;
            if (data === 'data: [DONE]') {
              dataDone = true;
              return;
            }
            this.loading = false;
            const json = JSON.parse(data.substring(6));
            this.articleContent += json.choices[0].text;
          });
          
          if (dataDone) {console.log(this.articleContent);break};
        }
    }
    ,
    markdownToHtml(markdownText) {
      const converter = new showdown.Converter({});
      const htmlContent = converter.makeHtml(markdownText);
      return htmlContent;
    }
  }

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}


.shadow_box {
  width: 80%;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
  border-radius: 5px;
}

label {
  display: block;
  margin-bottom: 10px;
  font-weight: bold;
}

input {
  width: 50%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
  margin-bottom: 20px;
}

button {
  background-color: #4CAF50;
  color: white;
  padding: 12px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  margin-bottom: 20px;
}

button:hover {
  background-color: #3e8e41;
}

.loading {
  font-size: 16px;
  text-align: center;
  color: #666;
}

div:last-of-type {
  margin-top: 20px;
  font-size: 18px;
  line-height: 1.5;
  color: #333;
}

.generate_text {
  text-align: left;
}
</style>
