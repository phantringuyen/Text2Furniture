# Text2Furniture
Tạo sinh cảnh nội thất ba chiều từ câu mô tả dựa vào trí tuệ nhân tạo tạo sinh.

## Chuẩn bị môi trường

Tạo môi trường conda ảo với câu lệnh sau

```
!conda create -n Text2Furniture python=3.10.12
!conda activate Text2Furniture
!pip install -r requirements.txt
```

Then install Pytorch3D by following the [official instructions](https://github.com/facebookresearch/pytorch3d/blob/main/INSTALL.md).
For example, to install Pytorch3D on Linux (tested with PyTorch 1.13.0, CUDA 12.2, Pytorch3D 0.7.7):

```
conda install -c fvcore -c iopath -c conda-forge fvcore iopath
pip install 'git+https://github.com/facebookresearch/pytorch3d.git@stable'
```

Download the pre-trained model weights for the text-to-image model and the fixed depth inpainting model:
https://drive.google.com/drive/folders/1z3pLvjndvZaiYveLY0Vo7ugZzKvl1O7V?usp=sharing


## Generate a Scene

As default, we generate a living room scene:

```python generate_scene.py```

Outputs are stored in ```Text2Furniture/output```.

### Generated outputs

We generate the following outputs per generated scene:

```
Mesh Files:
    <output_root>/fused_mesh/after_generation.ply: generated mesh after the first stage of our method
    <output_root>/fused_mesh/fused_final.ply: generated mesh after the second stage of our method
    <output_root>/fused_mesh/x_poisson_meshlab_depth_y.ply: result of applying poisson surface reconstruction on mesh x with depth y
    <output_root>/fused_mesh/x_poisson_meshlab_depth_y_quadric_z.ply: result of applying poisson surface reconstruction on mesh x with depth y and then decimating the mesh to have at least z faces
    
Renderings:
    <output_root>/output_rendering/rendering_t.png: image from pose t, that was rendered from the final mesh
    <output_root>/output_rendering/rendering_noise_t.png: image from a slightly different/noised pose t, that was rendered from the final mesh
    <output_root>/output_depth/depth_t.png: depth from pose t, that was rendered from the final mesh
    <output_root>/output_depth/depth_noise_t.png: depth from a slightly different/noised pose t, that was rendered from the final mesh

Metadata:
    <output_root>/settings.json: all arguments used to generate the scene
    <output_root>/seen_poses.json: list of all poses in Pytorch3D convention used to render output_rendering (no noise)
    <output_root>/seen_poses_noise.json: list of all poses in Pytorch3D convention used to render output_rendering (with noise)
    <output_root>/transforms.json: a file in the standard NeRF convention (e.g. see NeRFStudio) that can be used to optimize a NeRF for the generated scene. It refers to the rendered images in output_rendering (no noise).
```

We also generate the following intermediate outputs during generation of the scene:

```
    <output_root>/fused_mesh/fused_until_frame_t.ply: generated mesh using the content until pose t
    <output_root>/rendered/rendered_t.png: image from pose t, that was rendered from mesh_t
    <output_root>/mask/mask_t.png: mask from pose t, that signals unobserved regions
    <output_root>/mask/mask_eroded_dilated_t.png: mask from pose t, after applying erosion/dilation
    <output_root>/rgb/rgb_t.png: image from pose t, that was inpainted with the text-to-image model
    <output_root>/depth/rendered_depth_t.png: depth from pose t, that was rendered from mesh_t
    <output_root>/depth/depth_t.png: depth from pose t, that was predicted/aligned from rgb_t and rendered_depth_t
    <output_root>/rgbd/rgbd_t.png: combination of rgb_t and depth_t placed next to each other
```

### Create a scene from another room type

Generate indoor-scenes of arbitrary rooms by specifying another ```--trajectory_file``` as input:

```python generate_scene.py --trajectory_file model/trajectories/examples/bedroom.json```

We provide a bunch of [example rooms](model/trajectories/examples).


## Acknowledgements

Our work builds on top of amazing open-source networks and codebases. 
We thank the authors for providing them.

- [IronDepth](https://github.com/baegwangbin/IronDepth) [1]: a method for monocular depth prediction, that can be used for depth inpainting.
- [StableDiffusion](https://huggingface.co/stabilityai/stable-diffusion-2-inpainting) [2]: a state-of-the-art text-to-image inpainting model with publicly released network weights.

[1] IronDepth: Iterative Refinement of Single-View Depth using Surface Normal and its Uncertainty, BMVC 2022, Gwangbin Bae, Ignas Budvytis, and Roberto Cipolla

[2] High-Resolution Image Synthesis with Latent Diffusion Models, CVPR 2022, Robin Rombach, Andreas Blattmann, Dominik Lorenz, Patrick Esser, and Björn Ommer
