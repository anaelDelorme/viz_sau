<script lang="ts">
    /* The html content of the tag */
    export let html; //: string;
    /* regex: Regex that matches the parts of the html around which the component should be inserted
    *  component: The component to insert
    *  props: The props to pass to the component
    * */
    export let rules; //: {regex: RegExp, component: any}[];
    console.log("Rules : ", rules);

    let parsedHTML = [html];
    console.log("parsedHTML : ", parsedHTML);
    rules.forEach(rule => {
        console.log("rule : ", rule);
        parsedHTML = parsedHTML.map(substr => substr.split(rule.regex)).flat();
        console.log("parsedHTML 2 : ", parsedHTML);

    });
</script>

{#each parsedHTML as part}
    {@const match = (rules.find(rule => rule.regex.test(part)))}
    {console.log("MATCH", match)}
    {#if match}
        {console.log("MATCH component", match.component)}
        <svelte:component this={match.component}>
            {console.log("part:", part)}
            {@html part}
        </svelte:component>
    {:else}
        {@html part}
    {/if}
{/each}