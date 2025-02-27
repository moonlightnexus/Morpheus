<!--
SPDX-FileCopyrightText: Copyright (c) 2022-2023, NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

## DFP Deployment Module

This module function sets up modular Digital Fingerprinting Pipeline instance.

### Configurable Parameters

| Parameter           | Type | Description                               | Example Value | Default Value |
|---------------------|------|-------------------------------------------|---------------|---------------|
| `inference_options` | dict | Options for the inference pipeline module | See Below     | `[Required]`  |
| `training_options`  | dict | Options for the training pipeline module  | See Below     | `[Required]`  |

### Training Options Parameters

| Parameter                    | Type | Description                                    | Example Value        | Default Value |
|------------------------------|------|------------------------------------------------|----------------------|---------------|
| `batching_options`           | dict | Options for batching the data                  | See Below            | `-`           |
| `cache_dir`                  | str  | Directory to cache the rolling window data     | "/path/to/cache/dir" | `./.cache`    |
| `dfencoder_options`          | dict | Options for configuring the data frame encoder | See Below            | `-`           |
| `mlflow_writer_options`      | dict | Options for the MLflow model writer            | See Below            | `-`           |
| `preprocessing_options`      | dict | Options for preprocessing the data             | See Below            | `-`           |
| `stream_aggregation_options` | dict | Options for aggregating the data by stream     | See Below            | `-`           |
| `timestamp_column_name`      | str  | Name of the timestamp column used in the data  | "my_timestamp"       | `timestamp`   |
| `user_splitting_options`     | dict | Options for splitting the data by user         | See Below            | `-`           |

### Inference Options Parameters

| Parameter                    | Type | Description                                    | Example Value        | Default Value  |
|------------------------------|------|------------------------------------------------|----------------------|----------------|
| `batching_options`           | dict | Options for batching the data                  | See Below            | `-`            |
| `cache_dir`                  | str  | Directory to cache the rolling window data     | "/path/to/cache/dir" | `./.cache`     |
| `detection_criteria`         | dict | Criteria for filtering detections              | See Below            | `-`            |
| `fallback_username`          | str  | User ID to use if user ID not found            | "generic_user"       | `generic_user` |
| `inference_options`          | dict | Options for the inference module               | See Below            | `-`            |
| `model_name_formatter`       | str  | Format string for the model name               | "model_{timestamp}"  | `[Required]`   |
| `num_output_ports`           | int  | Number of output ports for the module          | 3                    | `-`            |
| `timestamp_column_name`      | str  | Name of the timestamp column in the input data | "timestamp"          | `timestamp`    |
| `stream_aggregation_options` | dict | Options for aggregating the data by stream     | See Below            | `-`            |
| `user_splitting_options`     | dict | Options for splitting the data by user         | See Below            | `-`            |
| `write_to_file_options`      | dict | Options for writing the detections to a file   | See Below            | `-`            |

### `batching_options`

| Key                      | Type            | Description                         | Example Value                               | Default Value              |
|--------------------------|-----------------|-------------------------------------|---------------------------------------------|----------------------------|
| `end_time`               | datetime/string | Endtime of the time window          | "2023-03-14T23:59:59"                       | `None`                     |
| `iso_date_regex_pattern` | string          | Regex pattern for ISO date matching | "\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}" | `<iso_date_regex_pattern>` |
| `parser_kwargs`          | dictionary      | Additional arguments for the parser | {}                                          | `{}`                       |
| `period`                 | string          | Time period for grouping files      | "1d"                                        | `D`                        |
| `sampling_rate_s`        | integer         | Sampling rate in seconds            | 0                                          | `None`                       |
| `start_time`             | datetime/string | Start time of the time window       | "2023-03-01T00:00:00"                       | `None`                     |

### `dfencoder_options`

| Parameter         | Type  | Description                            | Example Value                                                                                                                                                                                                                                                 | Default Value |
|-------------------|-------|----------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| `feature_columns` | list  | List of feature columns to train on    | ["column1", "column2", "column3"]                                                                                                                                                                                                                             | `-`           |
| `epochs`          | int   | Number of epochs to train for          | 50                                                                                                                                                                                                                                                            | `-`           |
| `model_kwargs`    | dict  | Keyword arguments to pass to the model | {"encoder_layers": [64, 32], "decoder_layers": [32, 64], "activation": "relu", "swap_p": 0.1, "lr": 0.001, "lr_decay": 0.9, "batch_size": 32, "verbose": 1, "optimizer": "adam", "scalar": "min_max", "min_cats": 10, "progress_bar": false, "device": "cpu"} | `-`           |
| `validation_size` | float | Size of the validation set             | 0.1                                                                                                                                                                                                                                                           | `-`           |

### `monitor_options`

| Key                         | Type    | Description                                                | Example Value | Default Value |
| ----------------------------|---------|------------------------------------------------------------|---------------|---------------|
| `description`               | string  | Name to show for this Monitor Stage in the console window  | "Progress"    | `Progress`    |
| `silence_monitors`          | bool    | Silence the monitors on the console                        | True     | `False`        |
| `smoothing`                 | float   | Smoothing parameter to determine how much the throughput should be averaged | 0.01 | `0.05` |
| `unit`                      | string  | Units to show in the rate value                             | "messages"    | `messages`    |
| `delayed_start`             | bool    | When delayed_start is enabled, the progress bar will not be shown until the first message is received. Otherwise, the progress bar is shown on pipeline startup and will begin timing immediately. In large pipelines, this option may be desired to give a more accurate timing. | True  | `False`   |
| `determine_count_fn_schema` | string  | Custom function for determining the count in a message      | "Progress"    | `Progress`    |
| `log_level`                 | string  | Enable this stage when the configured log level is at `log_level` or lower. | "DEBUG" | `INFO` |


### `mlflow_writer_options`

| Key                         | Type       | Description                       | Example Value                 | Default Value |
|-----------------------------|------------|-----------------------------------|-------------------------------|---------------|
| `conda_env`                 | string     | Conda environment for the model   | "path/to/conda_env.yml"       | `[Required]`  |
| `databricks_permissions`    | dictionary | Permissions for the model         | See Below                     | `None`        |
| `experiment_name_formatter` | string     | Formatter for the experiment name | "experiment_name_{timestamp}" | `[Required]`  |
| `model_name_formatter`      | string     | Formatter for the model name      | "model_name_{timestamp}"      | `[Required]`  |
| `timestamp_column_name`     | string     | Name of the timestamp column      | "timestamp"                   | `timestamp`   |

### `stream_aggregation_options`

| Parameter               | Type   | Description                                                 | Example Value | Default Value |
|-------------------------|--------|-------------------------------------------------------------|---------------|---------------|
| `cache_mode`            | string | The user ID to use if the user ID is not found              | "batch"       | `batch`       |
| `min_history`           | int    | Minimum history to trigger a new training event             | 1             | `1`           |
| `max_history`           | int    | Maximum history to include in a new training event          | 0             | `0`           |
| `timestamp_column_name` | string | Name of the column containing timestamps                    | "timestamp"   | `timestamp`   |
| `aggregation_span`      | string | Lookback timespan for training data in a new training event | "60d"         | `60d`         |
| `cache_to_disk`         | bool   | Whether or not to cache streaming data to disk              | false         | `false`       |
| `cache_dir`             | string | Directory to use for caching streaming data                 | "./.cache"    | `./.cache`    |

### `user_splitting_options`

| Key                     | Type | Description                                          | Example Value               | Default Value  |
|-------------------------|------|------------------------------------------------------|-----------------------------|----------------|
| `fallback_username`     | str  | The user ID to use if the user ID is not found       | "generic_user"              | `generic_user` |
| `include_generic`       | bool | Whether to include a generic user ID in the output   | false                       | `false`        |
| `include_individual`    | bool | Whether to include individual user IDs in the output | true                        | `false`        |
| `only_users`            | list | List of user IDs to include; others will be excluded | ["user1", "user2", "user3"] | `[]`           |
| `skip_users`            | list | List of user IDs to exclude from the output          | ["user4", "user5"]          | `[]`           |
| `timestamp_column_name` | str  | Name of the column containing timestamps             | "timestamp"                 | `timestamp`    |
| `userid_column_name`    | str  | Name of the column containing user IDs               | "username"                  | `username`     |

### `detection_criteria`

| Key          | Type  | Description                              | Example Value | Default Value |
|--------------|-------|------------------------------------------|---------------|---------------|
| `threshold`  | float | Threshold for filtering detections       | 0.5           | `0.5`         |
| `field_name` | str   | Name of the field to filter by threshold | "score"       | `probs`       |

### `inference_options`

| Parameter               | Type   | Description                                          | Example Value           | Default Value |
|-------------------------|--------|------------------------------------------------------|-------------------------|---------------|
| `model_name_formatter`  | string | Formatter for model names                            | "user_{username}_model" | `[Required]`  |
| `fallback_username`     | string | Fallback user to use if no model is found for a user | "generic_user"          | `generic_user`|
| `timestamp_column_name` | string | Name of the timestamp column                         | "timestamp"             | `timestamp`   |

### `write_to_file_options`

| Key                 | Type      | Description                              | Example Value   | Default Value    |
|---------------------|-----------|------------------------------------------|-----------------|------------------|
| `filename`          | string    | Path to the output file                  | "output.csv"    | `None`           |
| `file_type`         | string    | Type of file to write                    | "CSV"           | `AUTO`           |
| `flush`             | bool      | If true, flush the file after each write | false           | `false`          |
| `include_index_col` | bool      | If true, include the index column        | false           | `true`           |
| `overwrite`         | bool      | If true, overwrite the file if it exists | true            | `false`          |
