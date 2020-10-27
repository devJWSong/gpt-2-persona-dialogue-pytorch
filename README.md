# transformer-chatbot-pytorch

This is a multi-turn chatbot project using the **ReCoSa** structure introduced in *ReCoSa: Detecting the Relevant Contexts with Self-Attention for
Multi-turn Dialogue Generation*[[1]](#1).

The model detects the relevant dialogue histories with the self-attention mechanism, which uses the history-level transformer encoder, not the word-level.

The details of structure is as follows.

<img src="https://user-images.githubusercontent.com/16731987/97245796-2d7b6580-183f-11eb-9560-0c36038c0124.png" alt="The description of the ReCoSa structure." style="width: 60%; margin-left: 0;">

<br/>

---

### Configurations

You can set various hyperparameters by modifying `self.config` dictionary in `src/main.py` file.

The description of each variable is as follows. (Those not introduced in below table are set automatically and should not be changed.)

| Argument              | Type           | Description                                                  | Default                                                      |
| --------------------- | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `device`              | `torch.device` | The device type. (CUDA or CPU)                               | `torch.device('cuda') if torch.cuda.is_available() else torch.device('cpu')` |
| `learning_rate`       | `float`        | The learning rate.                                           | `5e-4`                                                       |
| `batch_size`          | `int`          | The batch size.                                              | `26`                                                         |
| `num_epochs`          | `int`          | The total number of iterations.                              | `20`                                                         |
| `max_len`             | `int`          | The maximum length of a sentence.                            | `300`                                                        |
| `num_heads`           | `int`          | The number of heads for Multi-head attention.                | `8`                                                          |
| `encoder_num_layers`  | `int`          | The number of layers in the encoder.                         | `6`                                                          |
| `decoder_num_layers`  | `int`          | The number of layers in the decoder.                         | `6`                                                          |
| `d_model`             | `int`          | The size of hidden states in the model.                      | `512`                                                        |
| `d_ff`                | `int`          | The size of intermediate  hidden states in the feed-forward layer. | `2048`                                                       |
| `dropout`             | `float`        | The dropout rate.                                            | `0.1`                                                        |
| `max_time`            | `int`          | The maximum length of the dialogue history to be attended.   | `20`                                                         |
| `nucleus_p`           | `float`        | The ratio of the probability mass for top-$p$ sampling(nucleus sampling). | `0.9`                                                        |
| `ckpt_dir`            | `str`          | The path for saved checkpoints.                              | `saved_model`                                                |
| `data_dir`            | `str`          | Name of the parent directory where data files are stored.    | `'data'`                                                     |
| `train_name`          | `str`          | The prefix of the train data files' name.                    | `train`                                                      |
| `valid_name`          | `str`          | The prefix of the validation data files' name.               | `validation`                                                 |
| `dialogue_split_line` | `str`          | The line for splitting each dialogue in the preprocessed data files. | `[END OF DIALOGUE]`                                          |
| `end_command`         | `str`          | The command to stop the conversation when inferencing.       | `Abort!`                                                     |
| `bos`                 | `str`          | The BOS(Beginning Of Sentence) token.                        | `<bos>`                                                      |
| `eos`                 | `str`          | The EOS(End Of Sentence) token.                              | `<eos>`                                                      |
| `pad`                 | `str`          | The padding token.                                           | `<pad>`                                                      |
| `gru_num_layers`      | `int`          | The number of layers in the word-level GRU.                  | `2`                                                          |
| `gru_dropout`         | `float`        | The dropout rate for GRU.                                    | `0.3`                                                        |

<br/>

<hr style="background: transparent; border: 0.5px dashed;"/>

### Datasets

By default, I propose the codes for downloading the datasets and preprocessing.

There are $4$ types of the default datasets as follows.

<br/>

- DailyDialog[[2](#2)]
- EmpatheticDialogues[[3](#3)]
- Persona-Chat[[4](#4)]
- BlendedSkillTalk[[5](#5)]

<br/>

You can use whatever data you want, but make sure that you should make `{data_dir}/{train_name}_id.txt` and `{data_dir}/{valid_name}_id.txt` 

consisting of token ids and dialogue split lines.

<img src="https://user-images.githubusercontent.com/16731987/97249049-6ff47080-1846-11eb-81bd-fce6b070db5b.png" alt="The details of the data format." style="width: 80%; margin-left: 0;">

<br/>

If you just run the download & preprocess script, you will also have `{data_dir}/{train_name}.txt` and `{data_dir}/{valid_name}.txt`.

But they are just for checking how the trimmed utterances look like, so they are not used for the actual training.

<br/>

<hr style="background: transparent; border: 0.5px dashed;"/>

### How to run

1. Install all required packages.

   ```shell
   pip install -r requirements.txt
   ```

   <br/>

2. Download & Preprocess all datasets. (If you want to use your own datasets, skip this step.)

   The variables `data_dir`, `train_name`, `valid_name`, `pad`, `bos`, `eos`, and `dialogue_split_line` must math with the configuration setting in `src/main.py`, otherwise it will cause error.

   ```shell
   python src/data_process.py
   ```

   Then there would be `{data_dir}` directory which has corresponding train & validation data files.

   In default setting, the structure of whole data directory should be like below.

   - `data`
     - `train_id.txt`
     - `train.txt`
     - `validation_id.txt`
     - `validation.txt`

   <br/>

3. Run the following command to train the model.

   ```shell
   python src/main.py --mode='train' --use_gpt=TRUE OR FALSE --ckpt_name=CHECKPOINT_NAME
   ```

   - `--mode`: You have to specify the mode among two options, 'train' or 'inference'.
   - `--use_gpt`: This determines whether the model uses the pre-trained GPT2's embedding layer. If it is `False`, then the embedding layer is trained from the beginning using `nn.Embedding`. (default: `False`)
   - `--ckpt_name`: This specify the checkpoint file name. This would be the name of trained checkpoint and you can continue your training with this model in the case of resuming training. If you want to conduct training from the beginning, this parameter should be omitted. When testing, this would be the name of the checkpoint you want to test. (default: `None`)

   <br/>

4. Run below command to conduct an inference with the trained model.

   ```shell
   python src/main.py --mode='test' --use_gpt=TRUE OR FALSE --ckpt_name=CHECKPOINT_NAME
   ```

   Obviously, `use_gpt` argument should match with the setting in training, in order to load a proper model.

   <br/>

---

### References

<a id="1">[1]</a> Zhang, H., Lan, Y., Pang, L., Guo, J., & Cheng, X. (2019). Recosa: Detecting the relevant contexts with self-attention for multi-turn dialogue generation. *arXiv preprint arXiv:1907.05339*. ([https://arxiv.org/abs/1907.05339](https://arxiv.org/abs/1907.05339))

<a id="2">[2]</a> Li, Y., Su, H., Shen, X., Li, W., Cao, Z., & Niu, S. (2017). Dailydialog: A manually labelled multi-turn dialogue dataset. *arXiv preprint arXiv:1710.03957*. ([https://arxiv.org/abs/1710.03957](https://arxiv.org/abs/1710.03957))

<a id="3">[3]</a> Rashkin, H., Smith, E. M., Li, M., & Boureau, Y. L. (2018). Towards empathetic open-domain conversation models: A new benchmark and dataset. *arXiv preprint arXiv:1811.00207*. ([https://arxiv.org/abs/1811.00207](https://arxiv.org/abs/1811.00207))

<a id="4">[4]</a> Zhang, S., Dinan, E., Urbanek, J., Szlam, A., Kiela, D., & Weston, J. (2018). Personalizing dialogue agents: I have a dog, do you have pets too?. *arXiv preprint arXiv:1801.07243*. ([https://arxiv.org/abs/1801.07243](https://arxiv.org/abs/1801.07243))

<a id="5">[5]</a> Smith, E. M., Williamson, M., Shuster, K., Weston, J., & Boureau, Y. L. (2020). Can You Put it All Together: Evaluating Conversational Agents' Ability to Blend Skills. *arXiv preprint arXiv:2004.08449*. ([https://arxiv.org/abs/2004.08449](https://arxiv.org/abs/2004.08449))

