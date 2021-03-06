<script lang="ts">
  import type { Activity, Timeframe } from "$src/types";
  import { getPreviousDurationText, toKebabCase } from "$src/utils";
  import { getContext, onMount } from "svelte";
  import type { Writable } from "svelte/store";
  import { CountUp } from "countup.js";

  //#region import assets
  import ellipsis from "$src/assets/icon-ellipsis.svg";
  import exercise from "$src/assets/icon-exercise.svg";
  import play from "$src/assets/icon-play.svg";
  import selfCare from "$src/assets/icon-self-care.svg";
  import social from "$src/assets/icon-social.svg";
  import study from "$src/assets/icon-study.svg";
  import work from "$src/assets/icon-work.svg";

  const imgs = {
    exercise,
    play,
    "self-care": selfCare,
    social,
    study,
    work,
  };
  //#endregion import assets

  export let activity: Activity;
  $: titleKebabCase = toKebabCase(activity.title);
  $: imageUrl = imgs[titleKebabCase];

  /** The current selected timeframe */
  let timeframe: Writable<Timeframe> = getContext("timeframe");
  $: previousDurationText = getPreviousDurationText($timeframe);
  $: currentDuration = activity.timeframes[$timeframe].current;
  $: previousDuration = activity.timeframes[$timeframe].previous;

  let currentDurationEl: HTMLSpanElement;
  let previousDurationEl: HTMLSpanElement;

  $: {
    if (currentDurationEl) {
      let currentCountUp = new CountUp(currentDurationEl, currentDuration);
      if (!currentCountUp.error) {
        currentCountUp.start();
      } else {
        console.error(currentCountUp.error);
      }
    }
  }

  $: {
    if (previousDurationEl) {
      let previousCountUp = new CountUp(previousDurationEl, previousDuration);
      if (!previousCountUp.error) {
        previousCountUp.start();
      } else {
        console.error(previousCountUp.error);
      }
    }
  }
</script>

<article
  class={["card", titleKebabCase].join(" ")}
  style:--imageUrl={`url(${imageUrl})`}
>
  <div class="card__content-wrapper">
    <h2 class="card__title">
      {activity.title}
    </h2>
    <div class="card__grab-handle" role="button">
      <img src={ellipsis} alt="" />
      <img src={ellipsis} alt="" />
    </div>
    <strong class="card__current-duration">
      <time datetime={`PT${currentDuration}H`}
        ><span bind:this={currentDurationEl} />hrs</time
      >
    </strong>
    <p class="card__previous-duration">
      {previousDurationText} &mdash;
      <time datetime={`PT${previousDuration}H`}
        ><span bind:this={previousDurationEl} />hrs</time
      >
    </p>
  </div>
</article>

<style lang="scss">
  $inline-padding: 1rem;
  $block-padding: 1.5em;

  .card {
    width: 100%;
    background: var(--bg-color) var(--imageUrl) no-repeat top -10px right $inline-padding;
    border-radius: $card-border-radius;
    padding-block-start: 2rem;

    &__content-wrapper {
      background: $clr-dark-blue;
      color: white;
      border-radius: $card-border-radius;
      padding: {
        block: $block-padding;
        inline: $inline-padding;
      }
      display: grid;
      align-items: start;
      row-gap: 0.5em;
      column-gap: 0.5em;
      height: 100%;
    }

    &__title {
      font-size: medium;
      font-weight: $fw-bold;
    }

    &__current-duration {
      font-size: xx-large;
      font-weight: $fw-light;
      line-height: 1;
    }

    &__previous-duration {
      font-size: small;
      font-weight: $fw-reg;
      text-align: right;
      color: $clr-pale-blue;
      grid-column: 2;
      align-self: center;
    }

    &__grab-handle {
      display: flex;
      flex-direction: column;
      justify-content: center;
      gap: 4px;
      width: 16px;
      grid-column: 2;
      justify-self: end;
      cursor: grab;

      &:active {
        cursor: grabbing;
      }
    }

    @media (min-width: $bp-tablet) {
      &__content-wrapper {
        padding: {
          inline: $inline-padding * 1.5;
        }
      }

      &__current-duration {
        margin-top: 0.4em;
        font-size: 3rem;
        grid-column: span 2;
      }

      &__previous-duration {
        grid-column: span 2;
        justify-self: start;
        text-align: left;
      }
    }
  }
  .work {
    --bg-color: #{$clr-work};
  }
  .play {
    --bg-color: #{$clr-play};
  }
  .study {
    --bg-color: #{$clr-study};
  }
  .exercise {
    --bg-color: #{$clr-exercise};
  }
  .social {
    --bg-color: #{$clr-social};
  }
  .self-care {
    --bg-color: #{$clr-self-care};
  }
</style>
