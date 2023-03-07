<script>
  import "katex/dist/katex.css";
  import "highlight.js/styles/github.css";
  import "bootstrap/dist/css/bootstrap.css";
  import MarkdownIt from "markdown-it";
  import MarkdownKatex from "markdown-it-katexx";
  import hljs from "highlight.js/lib/core";
  import hljsPlaintext from "highlight.js/lib/languages/plaintext";
  import hljsPython from "highlight.js/lib/languages/python";
  import mhl from "markdown-it-highlightjs";
  import { onMount } from "svelte";

  hljs.registerLanguage("plaintext", hljsPlaintext);
  hljs.registerLanguage("python", hljsPython);

  const DEFAULT_PROJECTS = [
    {
      name: "Default",
      text: "",
      headerLeft: "",
      headerRight: "",
    },
  ];

  let projects = DEFAULT_PROJECTS;
  onMount(() => {
    selected = "0";
    projects = JSON.parse(
      localStorage.getItem("sovaprojects") ?? JSON.stringify(DEFAULT_PROJECTS)
    );
    const interval = setInterval(() => {
      localStorage.setItem("sovaprojects", JSON.stringify(projects));
    }, 1000);
    return () => clearInterval(interval);
  });

  const md = MarkdownIt()
    .set({
      html: false,
      typographer: true,
      linkify: true,
    })
    .use(MarkdownKatex)
    .use(mhl, {
      auto: false,
      code: true,
      inline: false,
      hljs,
    });

  let selected = "0 ";
  let rendered = "";
  let err = null;

  let currentProject = projects[0];
  $: currentProject = projects[parseInt(selected)];
  $: try {
    rendered = md.render(currentProject.text);
    err = null;
  } catch (e) {
    err = e;
  }
</script>

<main>
  <div id="input">
    <p>
      <label
        >Project: <select bind:value={selected}>
          {#each projects as project, i}
            <option value={i}>{project.name}</option>
          {/each}
        </select></label
      >
      <button
        on:click={() => {
          projects = [...projects, { ...currentProject, name: prompt("Name:") }];
          selected = (projects.length - 1).toString();
        }}
      >
        mk
      </button>
      <button
        on:click={() => {
          projects.splice(parseInt(selected), 1);
          selected = (parseInt(selected) - 1).toString();
        }}
      >
        rm
      </button>
    </p>
    <p>
      <label
        >Header left: <input
          type="text"
          bind:value={currentProject.headerLeft}
        /></label
      >
    </p>
    <p>
      <label
        >Header right: <input
          type="text"
          bind:value={currentProject.headerRight}
        /></label
      >
    </p>
    <p>
      <textarea
        rows="20"
        bind:value={currentProject.text}
        style="width: 100%; height: 100%;"
      />
    </p>
  </div>
  <div id="header">
    <span>{@html currentProject.headerLeft}</span>
    <span style="text-align: right;">{@html currentProject.headerRight}</span>
  </div>
  <div>
    {@html rendered}
  </div>
  {#if err}
    <pre>{err.message}</pre>
  {/if}
</main>

<style>
  main {
    max-width: 60em;
    margin: 0 auto;
    padding: 1.5em;
  }
  @media print {
    #input {
      display: none;
    }
  }
  #header {
    display: flex;
    justify-content: space-between;
    align-items: end;
    margin: 0 -1em 1em;
    padding: 0 1em 0.5em;
    border-bottom: 1px solid black;
  }
  @media not print {
    #header {
      display: none;
    }
  }
</style>
