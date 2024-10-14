<script lang="ts">
  const word = "gamut".toUpperCase().split("");
  const letterCount = word.reduce((total, letter) => {
    total[letter] ? total[letter]++ : (total[letter] = 1);
    return total;
  }, {});
  let guess: string = "";
  let inputRef: HTMLInputElement;
  const wordLength = 5;

  let guesses: string[] = [];
  let colors: number[][] = [];

  // Game methods
  const checkGuess = (guess: string) => {
    // 0 for wrong, 1 for wrong place, 2 for right
    let result = Array(wordLength).fill(0);

    let count = { ...letterCount }; // create shallow copy, not reference

    console.log(count);

    // check both correct
    for (let [i, l] of word.entries())
      if (l == guess[i]) {
        count[l] -= 1;
        result[i] = 2;
        console.log("both correct");
      }
    // check wrong place, right letter
    for (let [i, l] of word.entries())
      if (result[i] < 2 && guess[i] in count && count[guess[i]] > 0) {
        count[guess[i]] -= 1;
        result[i] = 1;
        console.log("place wrong");
      }
    console.log(count);
    console.log(result);
    return result;
  };

  const handleWindowClick = () => {
    inputRef?.focus();
  };

  const handleSubmit = (e: Event) => {
    e.preventDefault(); // Prevents the form from refreshing the page

    if (guess.length < wordLength) return;

    guess = guess.toUpperCase();
    guesses = [...guesses, guess]; // to trigger reactivity
    colors = [...colors, checkGuess(guess)];
    guess = ""; // Clear guess
  };
</script>

<div
  class="flex flex-col justify-center items-center w-screen h-screen p-10 bg-black"
  on:click={handleWindowClick}
  role="button"
  tabindex="0"
  on:keydown={() => {}}
>
  <table class="border-separate border-spacing-[6px]">
    {#each { length: 6 } as _, i}
      <tr>
        {#each { length: wordLength } as _, j}
          <td
            class={`w-14 h-14 border-2 border-gray-700 font-bold text-white text-3xl text-center 
    ${
      colors[i] && colors[i][j] !== undefined
        ? colors[i][j] === 2
          ? "bg-green-700"
          : colors[i][j] === 1
            ? "bg-yellow-500"
            : "bg-gray-700"
        : ""
    }`}
          >
            {guesses.length > i ? guesses[i][j] : ""}
          </td>
        {/each}
      </tr>
    {/each}
  </table>

  <h1 class="font-bold text-white">{guess}</h1>
  <form on:submit={handleSubmit}>
    <input
      type="text"
      pattern="[A-Za-z]*"
      maxlength={wordLength}
      id="guess"
      bind:value={guess}
      bind:this={inputRef}
      class="w-0 h-0"
      required
    />
  </form>
</div>
