<p align="center">
  <img width="200" src="https://raw.githubusercontent.com/Billy-Sheppard/chart-js-rs/main/examples/favicon.png" alt="Material Bread logo">
</p>

# Chart.js types API in Rust 
[![crates.io](https://img.shields.io/crates/v/chart-js-rs.svg)](https://crates.io/crates/chart-js-rs)
[![docs.rs](https://docs.rs/chart-js-rs/badge.svg)](https://docs.rs/chart-js-rs)

***In Alpha, types added as needed, feel free to PR.***

## How to use

Check out the example folder for some code examples. The example uses WebAssembly and the [dominator](https://github.com/Pauan/rust-dominator) crate to produce charts. This library should be compatible with any WASM/HTML library.

The compiled webpage can be found here: https://billy-sheppard.github.io/chart-js-rs/examples/index.html

### Cargo.toml: 
```toml
[dependencies.chart-js-rs]
git = "https://github.com/Billy-Sheppard/chart-js-rs"
```

### Rust:
```rust
    let id = "[YOUR CHART ID HERE]";
    let chart = chart_js_rs::scatter::Scatter {
        id: id.to_string(),
        options: ChartOptions { .. },
        data: Dataset { .. },
        ..Default::default()
    };
    // to use any callbacks or functions you use render_mutate and refer to the JS below
    chart.to_chart().render_mutate();

    // else use render
    chart.to_chart().render();
```

### Your html file:
```html
<script src="https://cdn.jsdelivr.net/npm/chart.js@^4"></script>

...

<script type="module">
    import init from 'wasm.js';

    async function run() {
      await init();
    }

    run();
</script>

...

<script>
  // mutating charts
  function mutate_chart_object(v) { // must have this function name
    if (v.id === ("[YOUR CHART ID HERE]")) {
    // do any work here, this would prepend `$` to y1 axis tick labels
      v.options.scales.y1.ticks = {
        callback:
          function (value, _index, _values) {
            return '$' + value.toFixed(2);
          }
      };
    };

    return v
  }
</script>
```

<hr>

# Explainers

## Whats the difference between `my_chart.render()` and `mychart.render_mutate()`?
`.render()` is for simple charts, that don't require any further changes done using javascript code.

`.render_mutate()` allows for chart objects to be accessible in your javascript file, so you can mutate the object however required, especially useful for ChartJS functions not yet available in this library.

## How to use `struct FnWithArgs`?
`FnWithArgs` is a helper struct to allow serialization of javascript functions by encoding their body and arguments as a string. Then, as needed, the function can be rebuilt in JavaScipt, and called.

It is important then, that you know which variables are being parsed to the function. For this information, you can refer to the [Chart.js documentation](https://www.chartjs.org/docs/latest/).

`FnWithArgs` is used, for example, in implimenting conditional line segment colouring, according to the [docs](https://www.chartjs.org/docs/latest/samples/line/segments.html).
```rust
  Scatter::</*...*/> {
    data: {
      datasets: vec![
        Dataset {
          // ...
          segment: Segment {
            borderColor: 
              FnWithArgs::new() // Create the Function
                .arg("ctx") // Add a named arguement using a builder pattern
                .body("ctx.p0.parsed.y > ctx.p1.parsed.y ? 'red' : 'green'") // Add the function body, in this case make the line red if the slope is negative
          }
        }
      ]
    }
  }
```