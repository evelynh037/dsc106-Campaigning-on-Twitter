c<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Graph from "./Graph.svelte";
  import Categories from "./Categories.svelte";
  import Slogan from "./Slogan.svelte";
  import Donut from './Donut.svelte';
  let count, index, offset, progress;
 
  let slogans = [];

  let hillaryData = [];
  let trumpData = [];
  let common_words = [];

 
  let prefix = ""; 
  let inputText = "";
  let filtered_hillary;
  let filtered_trump;
 
 
  let suggest_words;
  let top_five_words;
  let if_empty = false;
  let selected_length = 0;
  let is_there = false;
  let selected=[];
  let selected_word = ["gun","border","women"];
  let selectedCategory = 'security_policy';

  //load data
  onMount(async () => {
    const res = await fetch('slogan_stats.csv');
    const csv = await res.text();
    slogans = d3.csvParse(csv, d3.autoType)
  })
  onMount(async () => {
    const res = await fetch('hillary_words.csv');
    const csv = await res.text();
    hillaryData = d3.csvParse(csv, d3.autoType)
  });
  onMount(async () => {
    const res = await fetch('trump_words.csv');
    const csv = await res.text();
    trumpData = d3.csvParse(csv, d3.autoType)
  });
  onMount(async () => {
    const res = await fetch('words_in_common.csv');
    const csv = await res.text();
    common_words = d3.csvParse(csv, d3.autoType)
  });
  //suggest words
  function handleClick(word) {
  if (selected.length <3 &&  (!(selected.includes(word)))){
    prefix = word;
    inputText = word;
    selected.push(prefix);
    selected_length+=1;
    selected_word = selected;
    is_there = false;
  } else if (selected.length <3 && selected.includes(word)) {
    is_there = true;
  }
  }
  function handleDelete(word){
    selected_word = selected_word.filter(selected => selected !== word)
    selected_length-=1;
    selected = selected_word;
  }
  //get frequency for input word
  $: filtered_hillary = selected_word.map(word => hillaryData.filter((d) => d.Word == word.toLowerCase()));
  $: filtered_trump = selected_word.map(word => trumpData.filter((d) => d.Word == word.toLowerCase()));
  //get five suggest words when typing in
  $: if (inputText !== "") {
    suggest_words = common_words.filter((d) => String(d.Word).startsWith(inputText.toLowerCase())).slice(0, 5);
    top_five_words = suggest_words.map(d => d.Word);
  } else{
    top_five_words = []
  }
  //check if words in data, and message if not
  $: if_empty = inputText !== "" && top_five_words.length === 0;
  //if clear inputbox, clear everything
  $: if (inputText == "") {
    prefix = ""
  }

  function scrollToSection(sectionId) {
    const section = document.getElementById(sectionId);
    if (section) {
      section.scrollIntoView({ behavior: 'smooth' });
    }
  }
 </script>
 
 
  <div class="left-panel">
    <div class=graph-container>
    <Donut {index}/>
    <Slogan {index}/>
    <Categories {index} {selectedCategory}/>
    <Graph {index} {filtered_hillary} {filtered_trump} {selected_word}/>
  </div>
  </div>
  <div class="right-panel">
    <Scroller
      top={0.0}
      bottom={1}
      threshold={0.5}
      bind:count
      bind:index
      bind:offset
      bind:progress
    >
      <div slot="foreground">
        <section class="centered-content"><br><br><br></section>
        <section id="scrollTargetSection" class="centered-content">
          <p>Presidential election has always been one of the most spotlighted events in most countries in the world. While many might take the voting procedure differently, the presidential election in the United States is conducted through an especially interesting system, in which two types of votings, <b>popular vote</b> and <b>electoral vote</b> are taken into evaluation. </p>
          <p>In the US Presidential Election, each state is given different amount of electoral vote based on its population. Then people do popular vote within each state. Whoever wins the popular vote of a state takes all of that state's electoral vote. The winner is whoever receive the most electoral votes.</p>
          <p>Now you might be thinking that since the amount of electoral vote per state is determined by its population, the winner of popular vote national wise is likely the winner of the electioin. Indeed, often times, that is what happened with US presidential election, but due to uncertainty within this system, sometimes it might not be the case...</p>
          <p>Let's look at one recent classic example of this... The 2016 presidential election between Hillary Clinton and Donald Trump.</p>
        </section>
        <section class="centered-content">
          <p>Notice the result of the two votes are different as shown in the chart on the left. While logically speaking this is a possible result, can we make any reasonable explanation as for why in this case, despite the disadvantage in overall popular vote, Trump still won the overall electoral vote? </p>
          <br>
          <p>Well, it turns out that one way to explain this is through analyzing what they put more focus on during their presidential campaign, that is, the content of their campaign.</p>
          <p>Twitter has played an increasingly prominent role in the 2016 US Presidential Election. Debates have raged and candidates have risen and fallen based on tweets. </p>
          <p>Does their campaign content affect their electoral results?</p>
          <p>This analysis is based on a dataset that provides approximately 3000 recent tweets from Hillary Clinton and Donald Trump, the two major-party presidential nominees. Follow along this article and get insights as to how the candidates' tweets refelct their electoral results.</p>
        </section>
        <section class="centered-content">
          <p>As the most symbolic campaign method, the use of slogan attracts public attention and give impression on the candidates that will influence their voting result. </p>
          <p>In this section, we will explore some of the most frequently mentioned slogans from each candidate and analyze their theme, the context in which they ususally appear, and the number of times they appeared in the tweets. </p>
          <p>Explore with the chart on the left, pay special notice to the description box on the top right to get hint on ways to continue exploration and possible interpretations. As you explore to the last layer, hover over the slogans for more detail.</p>
        </section>
        <section class="centered-content">
          <p><b>Are you able to draw any useful information out of the plot? Let's analyze it together now...</b><p>
          <p>Looking at the theme of the slogans, while Clinton's focus on building union with voters explains her advantage in popular vote, Trump's focus on using large quantity of imflammatory ambitious goals through patriotic lens definitely brings him more support as well, and constitute as part of the reason why Trump's popular vote does not fall behind too much as compared to Clinton's.</p>
        </section>
        <section class="centered-content">
          <p>Different groups cares about different political topics. Hence, which political issue the candidates draw their focus and campaigned on more than others has an impact on whether they will receive more votes from certain groups of people.</p>
          <p>In this section, we will explore and compare Trump's and Clinton's attention to different political topics through analyzing how often they brought up words relating to each category of political issues in their tweets.</p>
          <p>Use the plot on the left to explore number of times Trump or Clinton mentioned certain political issues more than the other candidate. Think about the underlying patterns.</p>
        </section>
        <section class="centered-content">
          <p><b>Now let's look at some of the patterns together and analyze how they might affect the election result.</b></p>
          <p>Pay attention to Clinton's attention to demographic issues like gender and minorities to win public votes among general publics in liberal states. Also noticing that Clinton has a lot of Spanish tweets and in fact, even opened an account for spanish tweets later, to get votes from Spanish speaking region like California and New Mexico. This might explain her victory in popular vote.</p>
          <p>However, Trump put special focus on serious matters like security issues and highly active with campaign advertisement, those might explain why he got a lot of popular votes too by attracting votes from patriotic groups and conservative states.</p>
          <p>Considering Clinton spent a great deal of focus on Spanish speaking region with high population, which are not swing states and was considerably democratic states already, that might explain why she fall behind in electoral votes in other states.</p>
        </section>
        <section class="centered-content">
          <div>
            <div class="centered-content">
            <p>Now that you have some idea about which topics each candidate spent more focus on. For instance, Clinton takes advantage of her identity and mentioned gender issue with words like "women" three times more than Trump to win supports from general publics with same identity.</p>
            <p>Type in a word to see how many time it appears in each nominees' tweet</p>
            <p>Remember, the frequency is out of ~3000 tweets for each of them. And is counting how many tweets contained the word of your choice </p>
            <p>Take your time to explore the topic you want! </p>
            <p>You've selected: </p>
            {#each selected_word as word}
            <button class="button-53" on:click={() => handleDelete(word)}>{word}</button>
            {/each}
            <br>
            <div class="input-container">
              <input type="text" bind:value={inputText} placeholder="Type in a word" />
            </div>
            {#if if_empty}
              <p>Try different word!</p>
            {/if}
            {#if selected_length == 3}
              <p>You have already choose three words. Let's explore the graph </p>
            {/if}
            {#if is_there == true && selected.includes(inputText)}
              <p>You already selected this, try different one! </p>
            {/if}
            {#if selected_length != 3}
            <div class="suggestions">
              {#each top_five_words as word}
                <button on:click={() => handleClick(word)}>{word}</button>
              {/each}
            </div>
            {/if}
          </div>
          </div>
        </section>
      </div>
    </Scroller>
  </div>
 
 
  <div class="header">
    <h1 class="sectionHeader">Campaigning on Twitter</h1>
    <p class="introText">Twitter has played an increasingly prominent role in the 2016 US Presidential Election. Debates have raged and candidates have risen and fallen based on tweets. This analysis is based on a dataset that provides approximately 3000 recent tweets from Hillary Clinton and Donald Trump, the two major-party presidential nominees. Follow along this article and get insights as to how the candidates' tweets refelct their electoral results.</p>
    <div class="scroll-down-container" on:click={() => scrollToSection('scrollTargetSection')}>
      <div class="arrow arrow1" ></div>
      <p class="scroll-message">Start Explore</p>
    </div>
  </div>
 
 
 <style>
  .header {
    position: absolute;
    left: 0;
    width: 101vw;
    height: 105%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-image: linear-gradient(to bottom, rgba(24, 106, 178, 0.7), rgba(80, 89, 98, 0.7)), url('https://graphics.wsj.com/elections/2016/donald-trump-hillary-clinton-on-the-economy/img/trumpVSclinton.jpg');
    background-size: cover;
    background-position: center;
    color: #ffffff;
    z-index: 99;
    top: 0px;
}

 
 
  .sectionHeader {
    font-size: 3.5em;
    margin: 0;
    text-align: center;
  }
  .introText {
    font-size: 1em;
    max-width: 50%;
  }
 
 
 .left-panel {
  left:0;
  top:0;
  position:fixed;
  height:100vh;
  width: 65%;
  background-color: rgba(105, 150, 193, 0.502);
 }
 .right-panel {
    position: absolute;
    right: 0;
    top:60vh;
    width: 36%;
    margin-right: -1vw;
    background-color: #ffffff80;
  }
 
 
  section {
    min-height: 110vh;
    color: black;
    background-color: #4ba9e380;
    background-image: linear-gradient(to top, rgba(123, 186, 237, 0.7), rgba(185, 215, 242, 0.625), rgba(117, 185, 241, 0.7));
    background-size: cover;
    background-position: center;
  }
 
 
  .centered-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
 }
  p {
    font-family: "Times New Roman", Times, serif;
    line-height: 2;
    max-width: 70%;
  }
  .suggestions {
  display: flex;
  flex-direction: column;
 }
 .suggestions button {
  width: 230px;
  height: 25px;
  background-color: rgb(243, 243, 251);
  background-color: #f0f0f0;
  border: 0.7px solid #d0d0d0e2;
  font-size: 15px;
  border-radius: 0.2em;
 }
 .suggestions button:hover {
  opacity: 0.7;
 }
 
 
 .graph-container {
  margin-top: 160px;
 }
 input {
  width: 250px;
  border: 1px solid #ccc;
  padding: 0px;
  font-size: 15px;
  transition: width 0.2s;
  border-radius: 1em;
 }
 
 
 .button-53 {
  background-color: #3DD1E7;
  border: 0 solid #E5E7EB;
  box-sizing: border-box;
  color: #000000;
  display: flex;
  font-family: ui-sans-serif,system-ui,-apple-system,system-ui,"Segoe UI",Roboto,"Helvetica Neue",Arial,"Noto Sans",sans-serif,"Apple Color Emoji","Segoe UI Emoji","Segoe UI Symbol","Noto Color Emoji";
  font-size: 1rem;
  font-weight: 700;
  justify-content: center;
  line-height: 1.75rem;
  padding: .75rem 1.65rem;
  position: relative;
  text-align: center;
  text-decoration: none #000000 solid;
  text-decoration-thickness: auto;
  width: 50%;
  max-width: 200px;
  position: relative;
  cursor: pointer;
  transform: rotate(-2deg);
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
 }
 
 
 .button-53:focus {
  outline: 0;
 }
 
 
 .button-53:after {
  content: '';
  position: absolute;
  border: 1px solid #000000;
  bottom: 4px;
  left: 4px;
  width: calc(100% - 1px);
  height: calc(100% - 1px);
 }
 
 
 .button-53:hover:after {
  bottom: 2px;
  left: 2px;
 }
 
 
 @media (min-width: 768px) {
  .button-53 {
    padding: .75rem 3rem;
    font-size: 1.25rem;
  }
 }

 @keyframes waveFlash {
  0%, 100% {
    border-color: white;
  }
  50% {
    border-color: hsla(0, 0%, 0%, 0); /* Change to your desired flashing color */
  }
}

 .arrow {
  width: 40px;
  height: 40px;
  border: solid white;
  border-width: 0 6px 6px 0;
  display: inline-block;
  margin: 0 10px;
  cursor: pointer;
  animation: waveFlash 1s infinite;
}
.scroll-down-container {
  position: absolute;
  bottom: 50px;
  left: 50%;
  transform: translateX(-50%);
  text-align: center;
}
.arrow1 {
  transform: rotate(45deg);
}
.scroll-message {
  color: white;
  font-size: 20px;
  margin-top: 10px;
  white-space: nowrap;
}
 </style>
 
 
 