# CSS Animations: Limitations & Suggestions

>  Every route you can take has advantages and disadvantages at same time. "Best way" is subjective.  Although those tips may not be the best way for everyone; they will lead to one well balanced point which will make the implementation easier to extend, the "trial and error" phase of defining animations less confusing and the performance way higher. 
>
> Feel free to create issues and pull requests.

## General Tips

- [ ] `animation-delay` only effects the first iteration of an animation. Sequent iterations are being triggered right after the previous one. If a delay between iterations is needed, solution can be stretching the animation until it is long enough to last until sequent iteration starts.  

  For example; an animation which is 2s long and needs 3s delay (between each iterations) can be defined like that:

  ```html
  <style>
    @keyframes my-anim {
      0% { /* start */ }
      40%, 100% { /* finish */ }
    }
    .my-class {
      animation: 5s my-anim; 
    }
  </style>
  ```

- [ ] Better layout can make easier the design of animation. Pick an initial point for every element and position them at there with using CSS (not in keyframes). For example; if a rectangle is intented to circle around `10px` away from the point `top: 3px; left: 5px`, initial point of that element should be located at (3, 5), that way the animation can be easily defined as:

  ```html
  <style>
    @keyframes my-anim {
      0% { transform: rotateZ(0deg) translateX(10px); }
      100% { transform: rotateZ(360deg) translateX(10px); }
    }
    .my-class {
      animation: 5s my-anim; 
      top: 3px;
      left: 5px;
    }
  </style>
  ```

- [ ] Functions that are given to `transform` property can be chained. Browser executes functions from right to left but if you think steps from origin to target point you can write them from left to right. If you want the element in previous example to be staying upright throughout its movement (10px away from (3, 5)), you can specify second rotation with opposite direction after the `translate` function.  

  ```html
  <style>
    @keyframes my-anim {
      0% { transform: rotateZ(0deg) translateX(10px) rotateZ(0deg); }
      100% { transform: rotateZ(360deg) translateX(10px) rotateZ(-360deg); }
    }
  </style>
  ```

- [ ] Even if the assets are vector based (like SVG), they can be pixelate throughout the animation if they scaled up. So, define the element with its peak appearence at start and if its necessary scale it down in declaration of animation. If the element have to be rendered smaller before animation start; `animation-fill-mode: both|forwards|backwards` can be used.  

	[animation-fill-mode in MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode)
	
- [ ] If you can abstract the requirements of responsive design with your layout, definitions of animations will be simpler.

- [ ] If overall frame consists of elements with similar behaviour, maybe you can use CSS variables for specifying the differences between them.

  ```html
  <style>
    @media not screen and (min-height:100vw) {
      --distance-vertical: calc(200px + 10vw);
    }
    @media only screen and (min-height:100vw) {
      --distance-vertical: 100px;
    }
    
    .my-class {
      top: calc(var(--row) * var(--distance-vertical));
      left: calc(var(--column) * var(--distance-horizontal));
    }
  </style>
  
  <div class="my-class" style="--row:1;--col:-2;"></div>
  ```

- [ ] When the overall frame consists of individually animated elements, the choreography can get out-of-sync so easily with performance penalties. When synchronization is necessary, define those elements within a HTML hierarchy, and apply common movements to parent elements. But if choreography is not needed, avoid hierarchy, since it makes the overall frame very specific (hierarchy is good for optimization but against flexibility).

- [ ] Browser won't re-calculate the relative distances (if they used in animation definition) until animation stopped. If the user changes the orientation of device; that will break the choreography, and will make elements zappy. As solution, create different animations for orientations/viewport resolutions, that way, orientation change won't break distances in animations. That solution doesn't solve the problem caused by resizing the browser window, because it produce continous viewport sizes.

## Performance Tips

- [ ] Functions specified with `transform` property doesn't force browser to re-layout every element in the page and actually, since they are executed by the GPU (not CPU) they are very performant. 
- [ ] Try to not use any property except `transform` and `opacity` for continous changes. If anything else has to be used; think again if there is another way, a trick maybe. 
- [ ] If `border-width` is needed to be changed throughout the animation, there is more performant way: masking. For the shape needed to be thinner/thicker on its border, create another element that has same shape and overlaps the original element with its `z-index`. Make its `background-color` same with background of frame. In this way, there is significant saving in performance.

- [ ] If any other property than `transform` have to be used and if it doesn't necessarily to be continously updated, try split those properties from main animation and define that animation's `animation-timing-function` property with `step(n, start|end)`. That way browser won't need constantly update the element for those expensive properties. For example if animation needs all elements in the frame to overlap others with order, you may need to use z-index in the animation definition. But it doesn't have to be re-calculated continously.

  [animation-timing-function at MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timing-function)  

	```html	
  <style>
    @keyframes my-anim-continous {
      0% { transform: /* ... */ translateX(-50px); /* ... */ }
      100% { transform: /* ... */ translateX(50px); /* ... */ }
    }

    @keyframes my-anim-stepped {
      0% { z-index: 4; }
      100% { z-index: 1; }
    }

    .my-class {
      animation: 2s my-anim-continous ease-in-out, 2s my-anim-stepped step(4, start);
      animation-delay: calc(2s / 4 * var(--element-index));
    }
  </style>
  
  <div class="my-class" style="--element-index:0;"></div>
  <div class="my-class" style="--element-index:1;"></div>
  <div class="my-class" style="--element-index:2;"></div>
  <div class="my-class" style="--element-index:3;"></div>
  ```


