<script>
  import copy from "../data/copy.json";
</script>

<div class="methods">
  <h3>Methodology</h3>
  <p>{copy.methods1}</p>
  <p>{copy.methods2}</p>
  <p>{copy.methods3}</p>
  <p>{copy.methods4}</p>
</div>
<div class="line-horiz">
  <div class="mid-line"></div>
</div>

<style>
  h3 {
    font-family: var(--narrow);
    text-align: center;
    text-transform: uppercase;
    font-weight: 700;
    color: var(--text-main);
  }

  .methods {
    width: calc(100% - 2rem);
    max-width: 80rem;
    border-left: 3px solid var(--jukebox-brown);
    border-right: 3px solid var(--jukebox-brown);
    position: flex;
    margin: 0 auto;
    font-family: var(--narrow);
    background-color: var(--jukebox-light-tan);
    padding: 2rem 3rem 2rem 3rem;
  }

  .line-horiz {
    width: 100%;
    border-top: 3px solid var(--jukebox-brown);
    border-bottom: 3px solid var(--jukebox-brown);
    height: 1.5rem;
    display: flex;
    align-items: center;
  }

  .mid-line {
    width: 100%;
    border-top: 3px solid var(--jukebox-brown);
  }

   p {
     max-width: 40rem;
     margin: 1rem auto;
     color: var(--text-main);
   }

   @media only screen and (max-width: 700px) {
    .methods {
      width: 100%;
      padding: 2rem 1rem 4rem 1rem;
      border-left: none;
      border-right: none;
    }
  }
</style>
