<script>
  import { onMount } from 'svelte';
  
  export let csvFilePath;
 console.log("csvfile :",csvFilePath)
  let categories = [];
  let data = [];
  let title = '';

  onMount(async () => {
    const response = await fetch(csvFilePath);
    console.log("response :",response)
    const text = await response.text();
    console.log("text :",text)
    const rows = text.split('\n');

    // Parse the CSV
    rows.forEach(row => {
      const columns = row.split(',');
      if (columns.length >= 2) {
        categories.push(columns[0]);
        data.push(Number(columns[1]));
      }
    });
    console.log("data :",data)
    console.log("categories :",categories)
    // Assuming the title is the first element of the first row
    title = categories.shift();
    dispatch('dataLoaded', { categories, data });
  });
</script>