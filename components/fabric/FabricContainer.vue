<template lang="pug">

.fabric-container
  canvas(
  :class="canvasClass"
  id="fabric-canvas"
  )

</template>

<script>
  import { fabric } from 'fabric';

  export default {
    name: 'FabricContainer',

    data() {
      return {
        canvasId: 'fabric-canvas',
        canvasOptions: {
          width: 500,
          height: 500,
        },
        outOfBoundsAlert: false,
        shapes: {

          rect: {
            constructor: 'Rect',
            options: {
              left: 100,
              top: 100,
              fill: 'red',
              width: 50,
              height: 50,
              originX: 'center',
              originY: 'center',
            },
          },

          circle: {
            constructor: 'Circle',
            options: {
              radius: 20,
              fill: 'green',
              left: 300,
              top: 400,
              originX: 'center',
              originY: 'center',

            },
          },

          rect2: {
            constructor: 'Rect',
            options: {
              left: 300,
              top: 300,
              fill: 'blue',
              width: 100,
              height: 50,
              originX: 'center',
              originY: 'center',
            },
          },
        },
      };
    },

    computed: {
      canvasClass() {
        return {
          'border-alert': this.outOfBoundsAlert,
        }
      },

      shapesToBeConstructed() {
        return Object.keys(this.shapes);
      },

      axisPropertyNames() {
        return {
          x: 'left',
          y: 'top',
        };
      },
    },

    mounted() {
      this.setupCanvas();
    },

    methods: {
      setupCanvas() {
        const canvas = this.createCanvas(this.canvasId, this.canvasOptions);
        this.createShapes(canvas);

        const limiter = (e) => {
          this.transformationLimiter(e, canvas);
        };

        canvas.on({
          'object:moving': limiter,
          'object:rotating': limiter,
          'object:scaling': limiter,
        });
      },

      transformationLimiter(e, canvas) {
        const object = e.target;
        object.setCoords();
        const pointsOutOfBounds = this.isAnyPointOutOfBounds(canvas, object);

        if (pointsOutOfBounds.length === 0) {
          this.outOfBoundsAlert = false;
          return;
        }

        this.outOfBoundsAlert = true;

        const boundaryCoords = this.getBoundaryCoords(canvas, object);

        for (const point of pointsOutOfBounds) {
          console.log('Out of bounds');
          const { axis, side } = point;
          const pointProperty = this.axisPropertyNames[axis];
          object.set(pointProperty, boundaryCoords[axis][side]);
        }

        object.set('scaleX', 1);
        object.set('scaleY', 1);
        object.set('angle', 0);
      },

      createCanvas(canvasId, canvasOptions) {
        return new fabric.Canvas(canvasId, canvasOptions);
      },

      createShape(shapeConstructor, shapeOption) {
        return new fabric[shapeConstructor](shapeOption);
      },

      createShapes(destinationCanvas) {
        for (const shape of this.shapesToBeConstructed) {
          const shapeReference = this.shapes[shape];
          const shapeInstance = this.createShape(shapeReference.constructor, shapeReference.options);
          destinationCanvas.add(shapeInstance);
        }
      },

      getBoundaryCoords(canvas, object) {
        const { width: objectWidth, height: objectHeight} = object;
        return {
          x: {
            min: objectWidth / 2,
            max: canvas.getWidth() - (objectWidth / 2),
          },
          y: {
            min: objectHeight / 2,
            max: canvas.getHeight() - (objectHeight / 2),
          }
        };
      },

      getObjectCoordinatePointNames(coordinatesObject) {
        const allPointsNames = Object.keys(coordinatesObject);
        const excludedPointName = 'mtr';
        return allPointsNames.filter(pointName => pointName !== excludedPointName);
      },

      isAnyPointOutOfBounds(canvas, object) {
        const pointNames = this.getObjectCoordinatePointNames(object.oCoords);
        const maximumX = canvas.getWidth() - (object.width / 2);
        const maximumY = canvas.getHeight() - (object.height / 2);
        const pointsOutOfBoundaries = [];
        for (const pointName of pointNames) {
          const point = object.oCoords[pointName];
          if (point.x <= 0) {
            pointsOutOfBoundaries.push({
              axis: 'x',
              side: 'min',
            });
          }
          if (point.x >= maximumX) {
            pointsOutOfBoundaries.push({
              axis: 'x',
              side: 'max',
            });
          }
          if (point.y <= 0) {
            pointsOutOfBoundaries.push({
              axis: 'y',
              side: 'min',
            });
          }
          if (point.y >= maximumY) {
            pointsOutOfBoundaries.push({
              axis: 'y',
              side: 'max',
            });
          }
        }
        return pointsOutOfBoundaries;
      }

    }
  }
</script>

<style scoped lang="scss">

  .fabric-container {
    display: flex;
    justify-content: center;
  }

  #fabric-canvas {
    border: 1px solid gray;

    &.border-alert {
      border-color: orangered;
      box-shadow: 0 0 5px orangered;
    }
  }

</style>
