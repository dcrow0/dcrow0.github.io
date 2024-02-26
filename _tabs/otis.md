---
title: Otis
order: 3
---

<style>
img {
    height: auto;
    width: auto\9; /* ie8 */
}
</style>



A randomly selected photo of the best dog in the world:

{% include random_image.html %}

<script>
  function refresh(){
        window.location.reload("Refresh")
      }
</script>

<div>
<input type="image" class="fl" width="60%" id="randomImage" src="" alt="The best dog in the world." object-fit="scale-down" onClick="refresh(this)">
</div>
