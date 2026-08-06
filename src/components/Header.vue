<template>
  <nav class="primary header">
    <img src="@/assets/feather.svg" alt="JRScore" class="logo" />

    <div class="title-area">
      <h5>JRScore</h5>
      <small>Convertisseur MEI / SVG basé sur Verovio</small>
    </div>
  </nav>
</template>

<style scoped>
.header {
  padding: 0 1.5rem;
  min-height: 72px;
  display: flex;
  align-items: center;
  gap: 1rem;
}

.logo {
  width: 42px;
  height: 42px;
  object-fit: contain;
}

.title-area {
  display: flex;
  flex-direction: column;
  justify-content: center;
  line-height: 1.1;
}

.title-area h5 {
  margin: 0;
  font-weight: 800;
  letter-spacing: -0.03em;
  font-size: 1.35rem;
}

.title-area small {
  margin-top: 0.25rem;
  opacity: 0.65;
  font-size: 0.85rem;
  font-style: italic;
}
</style>
