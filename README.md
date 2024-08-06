# Text2Furniture
Tạo sinh cảnh nội thất ba chiều từ câu mô tả dựa vào trí tuệ nhân tạo tạo sinh.

## Cài đặt môi trường

Tạo môi trường conda ảo với câu lệnh sau

```
!conda create -n Text2Furniture python=3.10.12
!conda activate Text2Furniture
!pip install -r requirements.txt
```

Sau đó cài đặt thư viện Pytorch3D theo [hướng dẫn này](https://github.com/facebookresearch/pytorch3d/blob/main/INSTALL.md).
Ví dụ, để cài đặt thư viện Pytorch3D trên nền tảng Linux (xác thực theo cấu hình PyTorch 1.13.0, CUDA 12.2, Pytorch3D 0.7.7):

```
conda install -c fvcore -c iopath -c conda-forge fvcore iopath
pip install 'git+https://github.com/facebookresearch/pytorch3d.git@stable'
```

Tải xuống bộ trọng số mô hình tiền huấn luyện cho mô hình văn bản thành hình ảnh và mô hình điền ảnh dựa trên độ sâu cố định:
https://drive.google.com/drive/folders/1z3pLvjndvZaiYveLY0Vo7ugZzKvl1O7V?usp=sharing


## Bắt đầu tạo sinh mô hình ba chiều

Outputs are stored in ```Text2Furniture/output```.

### Kết quả tạo sinh

Chúng tôi tạo ra các đầu ra sau cho mỗi cảnh được tạo theo định dạng sau:

```
Dữ liệu Mesh ba chiều:
    <output_root>/fused_mesh/after_generation.ply: lưới được tạo ra sau giai đoạn đầu tiên của phương pháp của chúng tôi
    <output_root>/fused_mesh/after_generation_meshlab_depth_y.ply: kết quả của việc áp dụng phương pháp tái tạo bề mặt Poisson trên lưới Mesh x với độ sâu y
    <output_root>/fused_mesh/after_generation_meshlab_depth_y_quadric_z.ply: kết quả của việc áp dụng tái tạo bề mặt Poisson trên lưới Mesh x với độ sâu y và sau đó chia lưới thành ít nhất z mặt với mục đích giảm dung lượng lưới Mesh nhưng vẫn giữ được chất lượng của nó
    
Renderings:
    <output_root>/output_rendering/rendering_t.png: hình ảnh từ góc nhìn thứ t, được kết xuẩ từ lưới cuối cùng
    <output_root>/output_rendering/rendering_noise_t.png: hình ảnh từ một góc nhìn thứ t có nhiễu, được kết xuất từ ​​lưới cuối cùng
    <output_root>/output_depth/depth_t.png: hình ảnh độ sâu từ góc nhìn thứ t, được kết xuẩ từ lưới cuối cùng
    <output_root>/output_depth/depth_noise_t.png: hình ảnh độ sâu từ một góc nhìn thứ t có nhiễu, được kết xuất từ ​​lưới cuối cùng

Metadata:
    <output_root>/settings.json: tất cả tham số được sử dụng để tạo ra cảnh
    <output_root>/seen_poses.json: danh sách tất cả các góc nhìn trong quy ước thư viện Pytorch3D được sử dụng để hiển thị output_rendering (không có nhiễu)
    <output_root>/seen_poses_noise.json: danh sách tất cả các góc nhìn trong quy ước thư viện Pytorch3D được sử dụng để hiển thị output_rendering (có nhiễu)
    <output_root>/transforms.json: một tệp trong quy ước NeRF chuẩn (ví dụ: NeRFStudio) được sử dụng để tối ưu hóa NeRF cho cảnh được tạo.

```

Chúng tôi cũng tạo ra các đầu ra trung gian sau trong quá trình tạo cảnh:

```
    <output_root>/fused_mesh/fused_until_frame_t.ply: tạo lưới bằng cách sử dụng kết quả tạo sinh (ảnh và độ sâu) cho đến góc nhìn thứ t
    <output_root>/rendered/rendered_t.png: hình ảnh từ góc nhìn thứ t, được kết xuất đồ họa từ mesh_t
    <output_root>/mask/mask_t.png: mặt nạ từ góc nhìn thứ t, tượng trưng cho các vùng chưa được quan sát
    <output_root>/mask/mask_eroded_dilated_t.png: mặt nạ từ , sau khi áp dụng thuật toán xói mòn/giãn nở
    <output_root>/rgb/rgb_t.png: hình ảnh từ góc nhìn thứ t, được tạo sinh bởi mô hình tạo ảnh từ văn bản
    <output_root>/depth/rendered_depth_t.png: ảnh độ sâu từ góc nhìn thứ t, được kết xuất đồ họa từ mesh_t
    <output_root>/depth/depth_t.png: ảnh độ sâu từ góc nhìn thứ t, được dự đoán/căn chỉnh từ ảnh rgb_t và ảnh độ sâu rendered_depth_t
    <output_root>/rgbd/rgbd_t.png: sự kết hợp của rgb_t và depth_t được đặt cạnh nhau
```


## Tài liệu tham khảo

Công trình của chúng tôi được xây dựng dựa trên các mạng lưới và cơ sở mã nguồn mở.
Chúng tôi cảm ơn các tác giả đã cung cấp chúng.

- [IronDepth](https://github.com/baegwangbin/IronDepth) [1]: một phương pháp dự đoán độ sâu bằng góc nhìn đơn, có thể được sử dụng để vẽ độ sâu.
- [StableDiffusion](https://huggingface.co/stabilityai/stable-diffusion-2-inpainting) [2]: một mô hình chuyển đổi văn bản thành hình ảnh hiện đại với bộ trọng số được công bố công khai.
- [Text2Room](https://www.researchgate.net/publication/377428396_Text2Room_Extracting_Textured_3D_Meshes_from_2D_Text-to-Image_Models) [3]: một phương pháp tạo ra các lưới Mesh ba chiều có kết cấu theo tỷ lệ phòng trong nhà từ câu mô tả làm đầu vào, đồng thời đây là phương pháp cơ sở nghiên cứu của chúng tôi.

[1] IronDepth: Iterative Refinement of Single-View Depth using Surface Normal and its Uncertainty, BMVC 2022, Gwangbin Bae, Ignas Budvytis, and Roberto Cipolla

[2] High-Resolution Image Synthesis with Latent Diffusion Models, CVPR 2022, Robin Rombach, Andreas Blattmann, Dominik Lorenz, Patrick Esser, and Björn Ommer

[3] Text2Room: Extracting Textured 3D Meshes from 2D Text-to-Image Models, 2023 IEEE/CVF ICCV, 7875-7886. 10.1109/ICCV51070.2023.00727. Höllein, Lukas & Cao, Ang & Owens, Andrew & Johnson, Justin & Nießner, Matthias.
