## 1 调试

1- 查看
模型构建检查通过：
- feyolov8s.yaml 可被 Ultralytics 正常解析；
- FEM 已作为 model.0 接入 backbone 前端；
- TFE 已作为 model.12 和 model.16 接入 neck 两个 Upsample 后；
- Detect head 类别数为 12；
- 参数量 11.236M，接近论文 11.18M；
- GFLOPs 为 45.5，高于论文 29.59，后续需检查 FEM/SFA 实现和 FLOPs 统计口径。

