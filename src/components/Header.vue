<template>
  <nav class="primary header">
    <img src="@/assets/feather.svg" alt="JRScore" class="logo" />

    <div class="title-area">
      <h5>JRScore</h5>
      <small>Convertisseur MEI / SVG basé sur Verovio</small>
    </div>

    <div class="max"></div>

    <button class="circle transparent about-button" data-ui="#about-dialog">
      <i>info</i>
    </button>

    <dialog id="about-dialog" class="about-dialog">
      <h5>À propos de JRScore</h5>

      <div>
        <p>
          JRScore est un visualiseur et convertisseur de partitions MEI vers SVG, basé sur le moteur
          de gravure musicale
          <a class="link" href="https://www.verovio.org/" target="_blank">
            Verovio
            <i class="tiny">open_in_new</i> </a
          >, distribué sous licence
          <a class="link" href="https://www.gnu.org/licenses/lgpl-3.0.html" target="_blank">
            LGPLv3
            <i class="tiny">open_in_new</i> </a
          >.
        </p>

        <p>Construit avec Vue 3 et BeerCSS.</p>
      </div>

      <nav class="right-align no-space">
        <button class="transparent link" data-ui="#about-dialog">Fermer</button>
      </nav>
    </dialog>
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
  flex-shrink: 0;
}

.title-area {
  display: flex;
  flex-direction: column;
  justify-content: center;
  line-height: 1.1;
  min-width: 0;
}

.title-area h5 {
  margin: 0;
  font-weight: 800;
  letter-spacing: -0.03em;
  font-size: 1.35rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.title-area small {
  margin-top: 0.25rem;
  opacity: 0.65;
  font-size: 0.85rem;
  font-style: italic;
}

@media only screen and (max-width: 600px) {
  .title-area small {
    display: none;
  }
}

.about-button {
  flex-shrink: 0;
}

.about-dialog {
  max-width: 26rem;
}

.about-dialog a.link {
  font-weight: 600;
}

.about-dialog a.link i {
  vertical-align: middle;
  font-size: 0.9rem;
  margin-left: 0.15rem;
}
</style>
