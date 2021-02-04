<script>
  import { onMount } from 'svelte';
  import { data } from './data.js';

  // DOM Elements
  let slider, slideImage;

  // State
  let isDragging = false,
    startPos = 0,
    currentTranslate = 0,
    prevTranslate = 0,
    animationID = 0,
    currentIndex = 0;

  onMount(() => {
    const slides = document.querySelectorAll('.slide');
    slides.forEach((slide, index) => {
      // Touch event listeners
      slide.addEventListener('touchstart', touchStart(index));
      slide.addEventListener('touchend', touchEnd);
      slide.addEventListener('touchmove', touchMove);
      // Mouse event listeners
      slide.addEventListener('mousedown', touchStart(index));
      slide.addEventListener('mouseup', touchEnd);
      slide.addEventListener('mouseleave', touchEnd);
      slide.addEventListener('mousemove', touchMove);
    });

    // Event callbacks
    function touchStart(index) {
      return function (event) {
        currentIndex = index;
        startPos = getPositionX(event);
        isDragging = true;
        animationID = requestAnimationFrame(animation);
        slider.classList.add('grabbing');
      };
    }

    function touchEnd() {
      isDragging = false;
      cancelAnimationFrame(animationID);
      const movedBy = currentTranslate - prevTranslate;
      if (movedBy < -100 && currentIndex < slides.length - 1) currentIndex += 1;
      if (movedBy > 100 && currentIndex > 0) currentIndex -= 1;
      setPositionByIndex();
      slider.classList.remove('grabbing');
    }

    function touchMove(event) {
      if (isDragging) {
        const currentPosition = getPositionX(event);
        currentTranslate = prevTranslate + currentPosition - startPos;
      }
    }

    // Helper functions
    function getPositionX(event) {
      return event.type.includes('mouse')
        ? event.pageX
        : event.touches[0].clientX;
    }

    function animation() {
      setSliderPosition();
      isDragging && requestAnimationFrame(animation);
    }

    function setSliderPosition() {
      slider.style.transform = `translateX(${currentTranslate}px)`;
    }

    function setPositionByIndex() {
      currentTranslate = currentIndex * -window.innerWidth;
      prevTranslate = currentTranslate;
      setSliderPosition();
    }
  });
</script>

<svelte:window on:contextmenu|preventDefault|stopPropagation />

<div bind:this={slider} class="slider-container">
  {#each data as { product, price, src }}
    <div class="slide" on:dragstart={(e) => e.preventDefault()}>
      <h2>{product}</h2>
      <h4>{price}</h4>
      <img
        bind:this={slideImage}
        {src}
        alt={product}
        on:dragstart|preventDefault
      />
      <button class="btn">Buy Now</button>
    </div>
  {/each}
</div>

<style>
  .slider-container {
    height: 90vh;
    display: inline-flex;
    overflow: hidden;
    transform: translateX(0);
    transition: transform 0.3s ease-out;
    cursor: grab;
  }

  .slide {
    max-height: 100vh;
    width: 100vw;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 1rem;
    user-select: none;
  }

  .slide img {
    max-width: 100%;
    max-height: 50%;
    transition: transform 0.3s ease-in-out;
  }

  .slide h2 {
    font-size: 2.5rem;
    margin-bottom: 0rem;
  }

  .slide h4 {
    font-size: 1.3rem;
  }

  .btn {
    background-color: #444;
    color: #fff;
    text-decoration: none;
    padding: 1rem 1.5rem;
  }

  .grabbing .slide img {
    transform: scale(0.9);
  }
</style>
