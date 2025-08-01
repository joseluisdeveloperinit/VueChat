<template lang="">
    <div class="advertise">
        eeeeeeeeeee
    </div>
</template>
<script>
export default {
    
}
</script>
<style>
.advertise {
  background-color: red;
  width: 100%;
  margin: 0.5rem;
  padding: 1rem;
  text-align: center;
}
/* Oculta en móviles (ancho < 768px) */
@media (max-width: 768px) {
  .advertise {
    display: none;
  }
}

/* Desktop: ancho fijo */
@media (min-width: 769px) {
  .advertise {
    width: 200px;
  }
}
</style>