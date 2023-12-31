Dự án MultilingualMT-UET-KC4.0 là dự án open-source được phát triển bởi nhóm UETNLPLab.

# Setup
## Cài đặt công cụ Multilingual-NMT

**Note**:
Lưu ý:
Phiên bản hiện tại chỉ tương thích với python>=3.6
```bash
git clone https://github.com/an23122002/Translator.git
cd Translator
pip install -r requirements.txt

# Quickstart

```

## Bước 1: Chuẩn bị dữ liệu

The current project comes with pre-existing data located in data

```bash
cd data
```

## Bước 2: Huấn luyện mô hình

Để huấn luyện một mô hình mới **hãy chỉnh sửa file YAML config**:
Cần phải sửa lại file config en_vi.yml chỉnh siêu tham số và đường dẫn tới dữ liệu huấn luyện:

```yaml
# data location and config section
data:
  train_data_location: data/Train/train2023
  eval_data_location:  data/Dev/dev2023
  src_lang: .lo
  trg_lang: .vi
log_file_models: 'model.log'
lowercase: false
build_vocab_kwargs: # additional arguments for build_vocab. See torchtext.vocab.Vocab for mode details
#  max_size: 50000
  min_freq: 5
# model parameters section
device: cuda
d_model: 512
n_layers: 6
heads: 8
# inference section
eval_batch_size: 8
decode_strategy: BeamSearch
decode_strategy_kwargs:
  beam_size: 5 # beam search size
  length_normalize: 0.6 # recalculate beam position by length. Currently only work in default BeamSearch
  replace_unk: # tuple of layer/head attention to replace unknown words
    - 0 # layer
    - 0 # head
input_max_length: 200 # input longer than this value will be trimmed in inference. Note that this values are to be used during cached PE, hence, validation set with more than this much tokens will call a warning for the trimming.
max_length: 160 # only perform up to this much timestep during inference
train_max_length: 50 # training samples with this much length in src/trg will be discarded
# optimizer and learning arguments section
lr: 0.2
optimizer: AdaBelief
optimizer_params:
  betas:
    - 0.9 # beta1
    - 0.98 # beta2
  eps: !!float 1e-9
n_warmup_steps: 4000
label_smoothing: 0.1
dropout: 0.1
# training config, evaluation, save & load section
batch_size: 32
epochs: 20
printevery: 200
save_checkpoint_epochs: 1
maximum_saved_model_eval: 5
maximum_saved_model_train: 5

```

Sau đó có thể chạy với câu lệnh:

```bash
python -m bin.main train --model Transformer --model_dir $MODEL/lo-vi.model --config $CONFIG/lo_vi.yml
```

**Note**:
Ở đây:
- $MODEL là dường dẫn tới vị trí lưu mô hình. Sau khi huấn luyện mô hình, thư mục chứa mô hình bao gồm mô hình huyến luyện, file config, file log, vocab.
- $CONFIG là đường dẫn tới vị trí chứa file config

## Bước 3: Dịch 

Mô hình dịch dựa trên thuật toán beam search và lưu bản dịch tại `$your_data_path/translate.en2vi.vi`.

```bash
python -m bin.main infer --model Transformer --model_dir $MODEL/la-vi.model --features_file $your_data_path/test.lo --predictions_file $your_data_path/translate.lo2vi.vi
```

## Bước 4: Đánh giá chất lượng dựa trên điểm BLEU

Đánh giá điểm BLEU dựa trên multi-bleu

```bash
perl thrid-party/multi-bleu.perl $your_data_path/translate.en2vi.vi < $your_data_path/tst2012.vi
```

|        MODEL       | BLEU (Beam Search) |
| :-----------------:| :----------------: |
| Transformer (Base) |        25.64       |


## Chi tiết tham khảo tại 
[nmtuet.ddns.net](http://nmtuet.ddns.net:1190/)

## Nếu có ý kiến đóng góp, xin hãy gửi thư tới địa chỉ mail kcdichdangu@gmail.com

## Xin trích dẫn bài báo sau:
```bash
@inproceedings{ViNMT2022,
  title = {ViNMT: Neural Machine Translation Toolkit},
  author = {Nguyen Hoang Quan, Nguyen Thanh Dat, Nguyen Hoang Minh Cong, Nguyen Van Vinh, Ngo Thi Vinh, Nguyen Phuong Thai, Tran Hong Viet},
  booktitle = {https://arxiv.org/abs/2112.15272},
  year = {2022},
}
```