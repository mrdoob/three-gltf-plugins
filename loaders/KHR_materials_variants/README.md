# [Three.js](https://threejs.org) [GLTFLoader](https://threejs.org/docs/#examples/en/loaders/GLTFLoader) [KHR_materials_variants](https://github.com/KhronosGroup/glTF/tree/master/extensions/2.0/Khronos/KHR_materials_variants) extension

## How to use

```javascript
import * as THREE from 'path_to_three.module.js';
import {GLTFLoader} from 'path_to_GLTFLoader.js';
import GLTFMaterialsVariantsExtension from 'path_to_three-gltf-plugins/loaders/KHR_materials_variants/KHR_materials_variants.js';

const loader = new GLTFLoader();
loader.register(parser => new GLTFMaterialsVariantsExtension(parser));
loader.load(path_to_gltf_asset, async gltf => {
  scene.add(gltf.scene);
  const variants = gltf.userData.variants; // [variantName0, variantName1, ...]
  const variantName = variants[theIndexYouWantToUse];
  await gltf.functions.selectVariant(gltf.scene, variantName);
  render();
});
```

## Compatible Three.js revision

&gt;= r126dev

## API

If glTF file includes `KHR_materials_variants` extension, the result returned by the loader will have the following data and function.

**gltf.userData.variants**

`variants` is an array of variant names like `[variantName0, variantName1, ...]`.

**mesh.userData.variantMaterials**

```javascript
{
  variantName0: {
    material: null,
    gltfMaterialIndex: number
  },
  variantName1: {
    material: null,
    gltfMaterialIndex: number
  },
  ...
}
```

`variantMaterials` is an object whose key is variant name and whose value has an index to glTF material index and variant material instance cache. Variant material cache is saved in `selectVariant()` or `ensureLoadVariants()`. The cache is also used when exporting variant materials.

**gltf.functions.selectVariant(object: THREE.Object3D, variantName: string, doTraverse = true: boolean): Promise**

`selectVariant()` is a function to switch materials to the ones associated with `variantName`. Unless `doTraverse` is set to `false` the function traverses the children and applys the change to all the child objects, too. The returned Promise will be resolved when all the selected materials are ready. 

**gltf.functions.ensureLoadVariants(object: THREE.Object3D, doTraverse = true: boolean): Promise**

`ensureLoadVariants()` is a function to ensure all the variant materials are loaded and saved under `mesh.userData.variantMaterials` (See [Side effects](#Side-effects) below). Unless `doTraverse` is set to `false`, the function traverses the children and applys it to all the child objects, too. You are recommended to call this function before exporting objects with [GLTFExporter KHR_materials_variants plugin](../../exporters/KHR_materials_variants/#README.md)

## Side effects

* `gltf.functions.selectVariant()` saves an original (core-spec) material as `mesh.userData.originalMaterial` for each Mesh
* `gltf.functions.selectVariant()` saves a selected variant material as `mesh.userData.variantMaterials[variantName].material` for each Mesh. It is expected to be used when exporting with [GLTFExporter KHR_materials_variants plugin](../../exporters/KHR_materials_variants/#README.md).
* The plugin ensures unique name in variant names so if duplicated names exist the plugin renames them. If you want to export the extension with the original names you are required to write an exporter plugin restoring the names.

## Replace with or add your custom variant materials

You can replace with or add your custom variant materials in your application by setting your custom material instance to `mesh.userData.variantMaterials[variantName].material` like

```
mesh.userData.variantMaterials[customVariantName] = {material: customVariantMaterial};
```

You can omit `.gltfMaterialIndex` property and `customVariantName` doesn't have to be in `gltf.userData.variants`.

## Limitations

* `selectVariant()` doesn't have selective option. All the meshes under a scene switch their materials.
* `selectVariant()` doesn't have effect to meshes which are already removed from a scene
* This plugin may not work if a glTF node has camera, light, or other extension objects in addition to mesh. This limitation may be removed if [this suggestion](https://github.com/mrdoob/three.js/pull/19359#issuecomment-774487100) is accepted.
* `mesh.userData.variantMaterials` are not serialized by Three.js `toJSON()` method correctly because it doesn't support the serialization of Three.js objects under `.userData`.
