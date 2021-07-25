<script>
  import List, {Item, Separator, Text, PrimaryText, SecondaryText, Graphic} from '@smui/list';
  import Button, {Group, GroupItem, Label, Icon} from '@smui/button';
  import Menu from "@smui/menu";
  import Select, {Option} from '@smui/select';
  import { bookworm, query } from './stores.js';
  import AestheticSelector from './AestheticSelector.svelte';
  export let aesthetic;
  export let metrics = 
    {"WordsPerMillion" : "Uses per million words",
    "TextCount": "Number of texts"};
  

  /* 
  So--this is complicated. Different types of charts can allow
  different types of axis labels. 
  */

  const definition = {
    "linechart": {
      "required": {
        "x": "quantitative",
        "y": "metric",
      },
      "optional": {
        "color":  "nominal",
        "group": "nominal",
        "strokeDash": "nominal"
      }
    },
    "heatmap": {
      "required": {
        "x": "data",
        "y": "data",
        "color":  "metric",
      },
      "optional": {
        "size":  "metric"
      }
    },
    "barchart": {
      "required": {
        "x": "metric",
        "y": "data",
      },
      "optional": {
        "color": "nominal"
      }
    },    
    "streamgraph": {
      "required": {
        "x": "quantitative",
        "y": "metric",
      },
      "optional": {
        "color": "nominal"
      }
    },
    "pointchart": {
      "required": {
        "x": "metric",
        "y": "data",
      },
      "optional": {
        "color": "nominal",
        "size": "metric"
      }
    },
    "scatterplot": {
      "required": {
        "x": "any",
        "y": "any",
      },
      "optional": {
        "color": "any",
        "size": "metric"
      }
    }
  }

  const selectors = {}

  const ptypes = [...Object.keys(definition)]

  let optional = []

  function possible_dimensions(plottype, aesthetic) {
    let dims = [];
    for (let [aes, dtype] of Object.entries(definition[plottype].required)) {
      dims.push({aes, dtype, active: true, required: true})
    }
    for (let [aes, dtype] of Object.entries(definition[plottype].optional)) {
      dims.push({aes, dtype, active: aesthetic[aes] !== undefined, required: false})
    }
    return dims
  }

  function possible_encoding_fields(dimension, schema) {
    console.log({schema})
    if (schema) {

    }
    const {aes, dtype: data_purpose, active} = dimension;
    const possibilities = []
    if (data_purpose == "nominal" || data_purpose == "data") {
      possibilities.push({
        label: "Search",
        value: "Search"
      })
    }
    if (data_purpose == "metric") {
      for (let [k, v] of Object.entries(metrics)) {
         possibilities.push({label: v, value: k})
      }
    }
    for (let {name, dbname, dtype, description} of schema) {
      if (data_purpose == "data") {
        possibilities.push({label: name || dbname, value: dbname})
      }
      if (dtype=="character" && data_purpose == "nominal") {
        possibilities.push({label: name || dbname, value: dbname})
      }
      if (dtype.match(/uint|date/) && data_purpose == "quantitative") {
        possibilities.push({label: name || dbname, value: dbname})
      }
    }
    
    return possibilities
  }

  function update_dimensions(plottype) {
    let dimensions = []
    if (definition[plottype] !== undefined) {
        for (let [aes, dtype] of Object.entries(definition[plottype].required)) {
          const current = {aes, dtype, options: []}
          if (dtype ==  "metric") {

          }
          dimensions.push(current)
        }
      }
      return dimensions
  }

  let plottype = $query.plottype;

  let add_aes;
  $ : dimensions = update_dimensions(plottype)


  function set_aesthetic(key, value) {
    aesthetic[key] = value;
    aesthetic = aesthetic;
  }

  </script>

  <Select label="Plot type" bind:value={$query.plottype} >
    {#each ptypes as ptype (ptype)}
      <Option value={ptype}>{ptype}</Option>
    {/each}
  </Select>


<div id=aesthetics>
  {#await $bookworm.schema}
    ...
    {:then schema}
    {#each possible_dimensions(plottype, aesthetic) as dim (dim.aes)}
      {#if dim.active}
        <AestheticSelector 
            required={dim.required} 
            possibilities={possible_encoding_fields(dim, schema)} 
            bind:value={aesthetic[dim.aes]} 
            title={dim.aes}>
        </AestheticSelector>
      {/if}
    {/each}
    <div style="min-width: 100px;">
      <Button on:click={() => add_aes.setOpen(true)}><Icon class="material-icons">add</Icon></Button>
      <Menu bind:this={add_aes}>
        <List>
          {#each possible_dimensions(plottype, aesthetic) as dim (dim.aes)}
            {#if !dim.active}
              <Item on:SMUI:action={() => {dim.active = true; aesthetic[dim.aes] = "Search";}}><Text>{dim.aes}</Text></Item>
            {/if}
          {/each}
        </List>
      </Menu>
    </div>
  {/await}
</div>


<style>

div.aesthetics {
  display: flex;
  margin-top: 5px;
}

.mdc-select__selected-text {
  min-width: 100px;
}

</style>