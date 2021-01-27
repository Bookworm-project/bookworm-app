<script>
import { query_selector_all } from "svelte/internal";


import { dirty_components } from "svelte/internal";


  import List, {Item, Separator, Text, PrimaryText, SecondaryText, Graphic} from '@smui/list';
  import Button, {Group, GroupItem, Label, Icon} from '@smui/button';

  import Menu from "@smui/menu";
  import Select, {Option} from '@smui/select';
  export let plottype;
  export let aesthetic;
  export let schema;
  export let metrics;

  if (schema === undefined || schema.length == 0) {
    throw "Missing schema"
  }
 
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



  const ptypes = [...Object.keys(definition)]

  let optional = []

  function possible_dimensions(plottype, aesthetic) {
    let dims = [];
    for (let [aes, dtype] of Object.entries(definition[plottype].required)) {
      dims.push({aes, dtype, active: true})
    }
    for (let [aes, dtype] of Object.entries(definition[plottype].optional)) {
      dims.push({aes, dtype, active: aesthetic[aes] !== undefined})
    }
    return dims
  }

  function possible_encoding_fields(dimension, schema) {
    const {aes, dtype, active} = dimension;
    const possibilities = []
    if (dtype == "nominal" || dtype == "data") {
      possibilities.push({
        label: "Search",
        value: "Search"
      })
    }
    if (dtype == "metric") {
      for (let [k, v] of Object.entries(metrics)) {
         possibilities.push({label: v, value: k})
      }
    }
    for (let {name, dbname, type, description} of schema) {
      if (dtype == "data") {
        possibilities.push({label: name, value: dbname})
      }
      if (type=="character" && dtype == "nominal") {
        possibilities.push({label: name, value: dbname})
      }
      if (type=="integer" && dtype == "quantitative") {
        possibilities.push({label: name, value: dbname})
      }
    }
    
    return possibilities
  }

  function update_dimensions(plottype, aesthetic) {
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

  $: dimensions = update_dimensions(plottype)

  </script>

  <Select label="Plot type" bind:value={plottype} >
    {#each ptypes as ptype (ptype)}
      <Option value={ptype}>{ptype}</Option>
    {/each}
  </Select>

<div id=aesthetics>

{#each possible_dimensions(plottype, aesthetic) as dim (dim.aes)}
  {#if dim.active}
  <Select label={dim.aes} bind:value={aesthetic[dim.aes]} class="shaped-outlined" variant="outlined">
      {#each possible_encoding_fields(dim, schema) as opt}
        <Option invalid={opt.value == ""} value={opt.value}>{opt.label}</Option>
      {/each}
  </Select>
  {/if}
{/each}

</div>


<style>

div.aesthetics {
  display: flex;
  margin-top: 5px;
}

</style>