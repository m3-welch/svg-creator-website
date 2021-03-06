<template>
    <transition
    name="show-hide-delete-button"
    enter-active-class="animate__animated animate__fadeInDown animate__faster"
    leave-active-class="animate__animated animate__fadeOutUp animate__faster"
    >
    <div class="delete-button" v-if="showDeleteButton" @click="deleteShape()">
        Delete Selected Shape
        <i class="mi-delete"></i>
    </div>
    </transition>
    <div id="canvas" class="canvas">
        <canvas id="c"></canvas>
    </div>
</template>

<script>
import { fabric } from 'fabric'
export default {
    name: 'Canvas',
    data() {
        return {
            canvas_height: 0,
            canvas_width: 0,
            canvas: null,
            device: null,
            objects: [],
            showDeleteButton: false,
            empty_annotations: {
                osc_address: null,
                osc_args: null,
                min: null,
                max: null,
                init: null,
                incr: null,
            }
        }
    },
    mounted() {
        this.canvas_height = window.innerHeight
        this.canvas_width = window.innerWidth - 250

        this.drawGrid()
    },
    props: {
        custom_size: Object
    },
    watch: {
        custom_size: function (newVal, oldVal) {
            console.log(oldVal)
            this.drawCustomDevice(newVal.custom_width, newVal.custom_height)
        }
    },
    methods: {
        deleteShape() {
            var index = this.objects.indexOf(this.canvas.getActiveObject())

            if (index > -1) {
                this.objects.splice(index, 1)
            }

            this.canvas.remove(this.canvas.getActiveObject())
            this.showDeleteButton = false
        },
        changeDevice(device) {
            switch(device) {
                case "sensel-morph":
                    this.drawSenselMorph()
                    break
                case "roli-lightpad-block":
                    this.drawRoliLightpadBlock()
                    break
                case "custom":
                    this.showCustomSizeForm()
                    break
                case "none":
                default:
                    this.clearDevice()
                    break
            }
            this.objects.forEach(object => this.canvas.bringToFront(object))
        },
        drawSenselMorph() {
            this.clearDevice()
            // Sensel Morph is 23cm x 13cm | 1 grid sqaure is 1cm
            this.addSquare(23 * 50, 13 * 50, true, null, null, this.empty_annotations)
            this.device.name = 'sensel'
        },
        drawRoliLightpadBlock() {
            this.clearDevice()
            // Roli Lightpad Block is 15 LEDs x 15 LEDs | 1 grid square is 1 LED
            this.addSquare(15 * 50, 15 * 50, true, null, null, this.empty_annotations)
            this.device.name = 'lightpad'
        },
        showCustomSizeForm() {
            this.$emit('show-custom-size-form')
        },
        drawCustomDevice(width, height) {
            this.addSquare(width * 50, height * 50, true, null, null, this.empty_annotations)
        },
        clearDevice() {
            this.canvas.remove(this.device)
        },
        drawGrid() {
            var canvas =  new fabric.Canvas('c', { selection: false })
            canvas.setHeight(this.canvas_height)
            canvas.setWidth(this.canvas_width)
            var grid = 50

            for (var i = 0; i < (this.canvas_width / grid); i++) {
                canvas.add(new fabric.Line([i * grid, 0, i * grid, this.canvas_height], { stroke: '#ccc', selectable: false, excludeFromExport: true }))
                canvas.add(new fabric.Line([ 0, i * grid, this.canvas_width, i * grid], { stroke: '#ccc', selectable: false, excludeFromExport: true }))
            }
            
            canvas.on('object:moving', function(options) {
                options.target.set({
                    left: Math.round(options.target.left / grid) * grid,
                    top: Math.round(options.target.top / grid) * grid
                })
            })

            this.canvas = canvas

            const helperObj = new fabric.Object({excludeFromExport: true})    //abstract invisible object
            helperObj.set("selectable", false)         //so the user is not able to select and modify it manually
            this.canvas.add(helperObj)

            this.canvas.on("mouse:down", (options) => {
                if (options.target && this.objects.includes(options.target)) {
                    this.showDeleteButton = true
                } else {
                    this.showDeleteButton = false
                }
            })

            this.canvas.on("object:added", () => {
                //workaround - selecting all objects to enable object controls

                let objects = this.canvas.getObjects()
                var selection = new fabric.ActiveSelection(objects, {
                    canvas: this.canvas,
                })
                this.canvas.setActiveObject(selection)   //selecting all objects...
                this.canvas.discardActiveObject()        //...and deselecting them
                this.canvas.requestRenderAll()
            })
        },
        addControl(color, size, type, shape, annotations) {
            switch (shape) {
                case 'square':
                    this.addSquare(size * 50, size * 50, false, color, type, annotations)
                    break
                case 'circle':
                    this.addCircle(size * 50, size * 50, color, type, annotations)
                    break
                case 'ring':
                    this.addRing(size * 50, size * 50, color, type, annotations)
                    break
            }
        },
        addSquare(width = 100, height = 100, device = false, color, type, annotations) {
            var square = new fabric.Rect({
                left: 200,
                top: 50,
                width: width,
                height: height,
                fill: !device ? color : 'rgba(0, 0, 0, 0)',
                originX: 'left',
                originY: 'top',
                evented: true,
                transparentCorners: false,
                cornerStyle: 'circle',
                hasRotatingPoint: false,
                selectable: !device,
                inter_type: type,
                inter_osc_address: annotations.osc_address,
                inter_osc_args: annotations.osc_args,
                min: annotations.min,
                max: annotations.max,
                init: annotations.init,
                incr: annotations.incr,
            })

            square.setControlsVisibility({
                mtr: false,
            })

            this.canvas.add(square)

            if (!device) {
                this.canvas.setActiveObject(square)
                this.objects.push(square)
            } else {
                square.stroke = '#' + Math.floor(Math.random()*16777215).toString(16)
                square.strokeWidth = 5
                square.strokeUniform = true
                square.hoverCursor = 'default'
                this.device = square
                square.excludeFromExport = true
            }

            this.canvas.renderAll()
        },
        addCircle(width = 50, height = 50, color, type, annotations) {
            var circle = new fabric.Circle({
                left: 200,
                top: 200,
                width: width,
                height: height,
                fill: color,
                originX: "left",
                originY: "top",
                evented: true,
                transparentCorners: false,
                cornerStyle: 'circle',
                radius: width / 2,
                hasRotatingPoint: false,
                inter_type: type,
                inter_osc_address: annotations.osc_address,
                inter_osc_args: annotations.osc_args,
                min: annotations.min,
                max: annotations.max,
                init: annotations.init,
                incr: annotations.incr,
            })

            circle.setControlsVisibility({
                mtr: false,
            })

            this.canvas.add(circle)
            this.objects.push(circle)

            this.canvas.renderAll()
        },
        addRing(width = 50, height = 50, color, type, annotations) {
            var ring = new fabric.Circle({
                left: 200,
                top: 200,
                width: width,
                height: height,
                fill: "rgba(0, 0, 0, 0)",
                stroke: color,
                strokeWidth: width / 4,
                originX: "left",
                originY: "top",
                evented: true,
                transparentCorners: false,
                cornerStyle: 'circle',
                radius: width / 2.65,
                hasRotatingPoint: false,
                inter_type: type,
                inter_osc_address: annotations.osc_address,
                inter_osc_args: annotations.osc_args,
                min: annotations.min,
                max: annotations.max,
                init: annotations.init,
                incr: annotations.incr,
            })

            ring.setControlsVisibility({
                mtr: false,
            })

            this.canvas.add(ring)
            this.objects.push(ring)

            this.canvas.renderAll()
        },
        exportCanvasToJson() {
            return JSON.stringify(this.canvas.toJSON(
                [
                    'inter_type',
                    'inter_osc_address',
                    'inter_osc_args',
                    'min',
                    'max',
                    'init',
                    'incr'
                ]
            ))
        }
    }
}
</script>

<style scoped>

.canvas {
    position: absolute;
    left: 250px;
    top: 50px;
}

.delete-button {
    z-index: 99;
    position: absolute;
    top: 55px;
    width: 200px;
    margin-left: 900px;
    margin-right: auto;
    background-color: #EF476F;
    color: #F4F4F9;
    height: 25px;
    line-height: 25px;
    padding: 5px;
    text-align: center;
    border: 2px solid #2F4550;
    border-radius: 5px;
}

.delete-button:hover {
    background-color: #EF587F;
    cursor: pointer;
}

.delete-button:active {
    background-color: #DE698F;
    cursor: pointer;
}

.delete-button i {
    font-size: 25px;
    float: right;
}

</style>