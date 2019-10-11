<script>
  import { onMount, onDestroy } from "svelte";
  import Matter from "matter-js";

  const Engine = Matter.Engine,
    Body = Matter.Body,
    Composite = Matter.Composite,
    Composites = Matter.Composites,
    Common = Matter.Common,
    World = Matter.World,
    Bodies = Matter.Bodies;

  let height = 500;
  let width = 500;
  let balls = [];

  let engine = Engine.create();
  engine.world.gravity.y = 0;

  onMount(() => {
    const wallThickness = 100;

    // we need to add some "walls" (invisible) to the world so aur balls can bounce....
    World.add(engine.world, [
      Bodies.rectangle(
        width / 2,
        (-1 * wallThickness) / 2,
        width,
        wallThickness,
        {
          isStatic: true
        }
      ),
      Bodies.rectangle(
        width / 2,
        height + wallThickness / 2,
        width,
        wallThickness,
        {
          isStatic: true
        }
      ),
      Bodies.rectangle(
        width + wallThickness / 2,
        height / 2,
        wallThickness,
        height,
        {
          isStatic: true
        }
      ),
      Bodies.rectangle(
        (-1 * wallThickness) / 2,
        height / 2,
        wallThickness,
        height,
        {
          isStatic: true
        }
      )
    ]);

    updateWorld();
  });

  let animationFrame = null;

  onDestroy(() => {
    if (animationFrame) window.cancelAnimationFrame(animationFrame);
  });

  function updateWorld() {
    animationFrame = window.requestAnimationFrame(updateWorld);

    // let the Matter.js update the state of world
    Engine.update(engine, 1000 / 60); // 16.666ms is the default

    // pull new state of the world from Matter.js and use that information to update our SVG representation of the world
    updateDataFromWorld();
  }

  function updateDataFromWorld() {
    balls = Composite.allBodies(engine.world)
      .filter(b => b.label === "ball")
      .map(b => {
        return {
          x: b.position.x,
          y: b.position.y,
          r: b.circleRadius,
          id: b.id
        };
      });
  }

  function handleClick(e) {
    addBall(e.x, e.y);
    updateDataFromWorld();
  }

  function addBall(x, y) {
    console.log(`Adding ball to ${x}, ${y}`);

    const bodyOptions = {
      label: "ball",
      frictionAir: 0,
      friction: 0.0001,
      restitution: 1
    };
    const body = Bodies.circle(x, y, 20, bodyOptions);

    const forceMagnitude = 0.05 * body.mass;
    Body.applyForce(body, body.position, {
      x:
        (forceMagnitude + Common.random() * forceMagnitude) *
        Common.choose([1, -1]),
      y: -forceMagnitude + Common.random() * -forceMagnitude
    });

    World.add(engine.world, body);
  }
</script>

<style>
  svg {
    width: 500px;
    height: 500px;
  }
</style>

<svg>
  <rect
    x="0"
    y="0"
    width="{width}"
    height="{height}"
    fill="lightblue"
    on:click="{handleClick}"
  />

  {#each balls as ball}
    <circle cx="{ball.x}" cy="{ball.y}" r="{ball.r}" fill="black"></circle>
  {/each}
</svg>

<h1>Playing with Svelte, SVG and Matter.js</h1>
<p>Click into the blue box to add some balls.....</p>
