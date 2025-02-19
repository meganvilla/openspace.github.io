---
title: Model Loading
layout: default

grand_parent: Builders
parent: Models
nav_order: 3
---

# Breaking change in 0.17.0: Model Loading
The model loading in OpenSpace has been updated to support models with multiple parts, materials and textures. This update unfortunately means that previous asset files with models need to be updated. Note that this feature was added in release 0.17.0.

Previously a model was specified in an asset file as a <code>Geometry</code> with a <code>Type</code>, <code>GeometryFile</code> and a <code>ColorTexture</code>. Now it is just specified with a <code>GeometryFile</code>, there is no need for a <code>ColorTexture</code> since this information will be read from the model. This may mean that the models that have previously been used might need to be exchanged or updated to properly work with the new model loading system, more on that later.

## Examples
How the Renderable for the Juno spacecraft was specified before:
~~~
    Renderable = {
        Type = "RenderableModel",
        Geometry = {
            Type = "MultiModelGeometry",
            GeometryFile = model .. "/Juno.obj",
            ColorTexture =  textures .. "/gray.png"
        },
        ModelTransform = RotationMatrix,
        LightSources = assetHelper.getDefaultLightSources(sunTransforms.SolarSystemBarycenter.Identifier)
    },
~~~

How the Renderable for the Juno spacecraft is specified now:
~~~
    Renderable = {
        Type = "RenderableModel",
        GeometryFile = model .. "/Juno.obj",
        ModelTransform = RotationMatrix,
        LightSources = assetHelper.getDefaultLightSources(sunTransforms.SolarSystemBarycenter.Identifier)
    },
~~~

## RenderableModelProjection
This breaking change is also applied to the RenderableModelProjection, for example in the Rosetta profile the model for 67P Churymov-Gerasimenkou was previously specified as:
~~~
    Renderable = {
        Type = "RenderableModelProjection",
        Geometry = {
            Type = "MultiModelGeometry",
            GeometryFile = models .. "/67P_rotated_5_130.obj",
            ColorTexture = textures .. "/gray.jpg"
        },
        Projection ...
    },
~~~

Now it is specified as:
~~~
    Renderable = {
        Type = "RenderableModelProjection",
        GeometryFile = models .. "/67P_rotated_5_130.obj",
        Projection ...
    },
~~~

## Models
As you may have noticed there is no need to specify the ColorTexture as before. Instead this information will be read from the model itself, which may mean that your old models need to be updated or exchanged. This depends very much on what kind of model you are using but in general there are two different ways to update the old model:

* Embed the material or texture to the model.

* Define a material file (.mtl) and link it to your model.

Both of these could be done with any modeling software (such as Blender, Maya or 3ds Max). In both cases you need to make sure that you export the model with the materials or textures. In the second case you would get a material file (.mtl) that should be next to your model file. You can also do the second case manually as explained below. 

### Create your own material file for your model
It is possible to create your own material file and connect your model to the correct textures by editing the files in a text editor. Depending on how complex and large your model is this could take some time and effort.

1. Start by creating a new text file and rename its extension to .mtl instead of .txt (you might have to turn on visible file extensions in Windows for this).

2. Open your model file and material file in a text editor (such as Notepad, Notepad++ or Visual Studio Code).

3. In the model file, at the top you need to link the material file you just created with: <code>mtllib filename.mtl</code> replacing filename with the name of the material file you created in step 1.

4. In the model file there should be a long list of data, go to the line where the data shifts to <code>f</code>. A tip is to use the search function that most text editors have with Ctrl + f and search for <code>f</code>. Right before the first line of <code>f</code> insert the line: <code>usemtl materialName</code>. This tells the model that this part of the model should have this material.

5. If there are several different lists of <code>f</code> you repeat step 4 until all lists of <code>f</code> has a material. You can use different materials (change materialName) if you would like the different parts of the model to have different materials or textures. 

6. Your model file should look something like this at this point (example with two materials):
~~~
    # Header ...

    mtllib 0.mtl
    o 10_(ESP)_External_Stowage_Platform_1_z1_ext_01.000
    v 2.529284 -0.517977 3.555833
    v 2.559681 -0.517977 3.555833

    ...

    vn -0.6966 0.4229 0.5796
    vn -0.8944 0.2626 -0.3619
    s off
    usemtl ISS_03_dull
    f 246/1/1 247/2/1 248/3/1
    f 246/1/1 249/4/1 247/2/1

    ...

    vn -0.3987 -0.5324 0.7467
    vn -0.7104 -0.4799 0.5148
    s 1
    usemtl white
    f 1216/1216/491 1217/1217/492 1218/1218/493
    f 1216/1216/491 1219/1219/494 1217/1217/492

    ...
~~~

7. Switch to the material file. Here is where you define your materials and the textures. Create a new material with: <code>newmtl materialName</code>. Note that materialName should be the same as you specified in the model file in step 4. Then connect the material to a texture using: <code>map_Kd textureName.png</code>. Note that the path to the texture should be given relative to the material file.

8. If you specified several **different** materials in step 5 you will need to repeat step 7 for every new material. 

9. In the end your material file should look something like this (with two materials):
~~~
    newmtl ISS_03_dull
        map_Kd 0.png

    newmtl white
        map_Kd white_20.png
~~~

10. Make sure that your material file is in the same directory as the model file. Your model should now be textured in OpenSpace.

## Debugging your model
If your model does not show up in OpenSpace and you are sure that you have done everything right there is a new tool that you could use for debugging. In the asset file you can add an optional property for forcing invisible parts of the model to render. This forces any part of the model that is invisible (has no texture or color) to render. This property is called <code>ForceRenderInvisible</code>, for example the Juno spacecraft:

~~~
    Renderable = {
        Type = "RenderableModel",
        GeometryFile = model .. "/Juno.obj",
        ForceRenderInvisible = true,
        ModelTransform = RotationMatrix,
        LightSources = assetHelper.getDefaultLightSources(sunTransforms.SolarSystemBarycenter.Identifier)
    },
~~~

Any part of the model that is invisible will now be rendered with a bright and colorful pink and green chessboard pattern. This pattern will also be forced if OpenSpace encountered any Error while loading the material or texture for the model, even without the property. This could make it easier to identify errors with your model.

If this property is left out in the asset file and your model has invisible parts there will be an info message in the log making you aware that there might be something wrong with the model. However, just because a part of the model is invisible does not necessarily mean that something is wrong with the model. If you are aware that your model has invisible parts and want to suppress this info message you can set the property to false to keep the log cleaner.
