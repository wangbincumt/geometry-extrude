# Geometry Extrude

A small and fast JavaScript library to extrude 2D polygons and polylines to 3D meshes. It depends on [earcut](https://github.com/mapbox/earcut) to do triangulation.

## Features

+ Extrude polygons with holes.

+ Extrude polylines with specific line thickness.

+ Generate position/uv/normal/indices array.

+ Bevel style.

## Basic Usage

Install with npm

```
npm i geometry-extrude
```

Extrude a simple square with hole

```js
import {extrudePolygon} from 'geometry-extrude';
const squareWithHole = [
    [[0, 0], [10, 0], [10, 10], [0, 10]],
    // Hole
    [[2, 2], [8, 2], [8, 8], [2, 8]]
];
const {indices, position, uv, normal} = extrudePolygon(squareWithHole, {
    depth: 2
});
```

### Use with ClayGL

```js
const {indices, position, uv, normal} = extrudePolygon(squareWithHole);
const geometry = new clay.Geometry();
geometry.attributes.position.value = position;
geometry.attributes.texcoord0.value = uv;
geometry.attributes.normal.value = normal;
geometry.indices = result.indices;
```

### Use with ThreeJS

```js
const {indices, position, uv, normal} = extrudePolygon(squareWithHole);
const geometry = new THREE.BufferGeometry();
geometry.addAttribute('position', new THREE.BufferAttribute(position, 3));
geometry.addAttribute('normal', new THREE.BufferAttribute(normal, 3));
geometry.setIndex(indices);
```

### Use with regl

```js
const {indices, position, uv, normal} = extrudePolygon(squareWithHole);
const draw = regl({
    frag: `...`,
    vert: `...`,

    attributes: {
        position: regl.buffer(position),
        uv: regl.buffer(uv),
        normal: regl.buffer(normal)
    },

    elements: regl.elements({
        primitive: 'triangles',
        data: indices
    })
});
```

## Full API List

### extrudePolygon

```js
extrudePolygon(
    // polygons same with coordinates of MultiPolygon type geometry in GeoJSON
    // See http://wiki.geojson.org/GeoJSON_draft_version_6#MultiPolygon
    polygons: GeoJSONMultiPolygonGeometry,
    // Options of extrude
    opts: {
        // Can be a constant value, or a function.
        // Default to be 1.
        depth?: ((idx: number) => number) | number,
        // Size of bevel, default to be 0, which is no bevel.
        bevelSize?: number,
        // Segments of bevel, default to be 2. Larger value will lead to smoother bevel.
        bevelSegments?: number,
        // If has smooth side, default to be false.
        smoothSide?: boolean,
        // If has smooth bevel, default to be false.
        smoothBevel?: boolean,
        // Transform the polygon to fit this rect.
        // Will keep polygon aspect if only width or height is given.
        fitRect?: {x?: number, y?: number, width?: number: height?: number},
        // Translate the polygon. Default to be [0, 0]
        // Will be ignored if fitRect is given.
        translate?: ArrayLike<number>,
        // Scale the polygon. Default to be [1, 1]
        // Will be ignored if fitRect is given.
        scale?: ArrayLike<number>
    }
) => {
    indices: Uint16Array|Uint32Array,
    position: Float32Array,
    normal: Float32Array,
    uv: Float32Array,
    boundingRect: {x: number, y: number, width: number, height: number}
}
```

### extrudePolyline

```typescript
extrudePolyline(
    // polylines same with coordinates of MultiLineString type geometry in GeoJSON
    // See http://wiki.geojson.org/GeoJSON_draft_version_6#MultiLineString
    polylines: GeoJSONMultiLineStringGeometry,
    // Options of extrude
    opts: {
        ////// Extended from opts in extrudePolygon

        // Thickness of line, default to be 1
        lineWidth?: number,
        // default to be 2
        miterLimit?: number
    }
) => {
    indices: Uint16Array|Uint32Array,
    position: Float32Array,
    normal: Float32Array,
    uv: Float32Array,
    boundingRect: {x: number, y: number, width: number, height: number}
}
```

### extrudeGeoJSON

```typescript
extrudeGeoJSON(
    // Extrude geojson with Polygon/LineString/MultiPolygon/MultiLineString geometries.
    geojson: GeoJSON,
    // Options of extrude
    opts: {
        ////// Extended from opts in extrudePolygon

        // Can be a constant value, or a function with parameter of each feature in geojson.
        // Default to be 1.
        depth?: ((feature: GeoJSONFeature) => number) | number
        // Thickness of line, default to be 1
        lineWidth?: number,
        // default to be 2
        miterLimit?: number
    }
) => {
    // Same result with extrudePolygon
    polygon: Object,
    // Same result with extrudePolyline
    polyline: Object
}
```