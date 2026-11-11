# K2 Horizon on AWS Marketplace

Deploy K2 Horizon models through AWS Marketplace and Amazon SageMaker AI.

These examples deploy models in your AWS account and demonstrate chat, streaming, conversation history, tool calling and batch inference. You control the deployment and its lifecycle.

## Examples

| Model | Notebook | Python helpers |
| --- | --- | --- |
| K2 Horizon 7B FP8 | [SageMaker notebook](notebooks/k2-horizon-7b-fp8.ipynb) | [Deployment and inference helpers](examples/k2-horizon-fp8-sagemaker.py) |

The first example uses the [K2 Horizon 7B FP8 checkpoint](https://huggingface.co/IFM/K2-Horizon-7B-FP8). Its configurations are specific to that checkpoint; future model examples may use different precision, hardware and settings.

Every supported instance type serves a total context of 32,768 tokens with one active sequence, FP8 weights and BF16 activations and KV cache. The model package applies these settings for the GPU it starts on; you do not set container environment variables.

| SageMaker instance | GPU | Real-time | Batch |
| --- | --- | :---: | :---: |
| `ml.g6.2xlarge` | 1 NVIDIA L4 | Yes | Yes |
| `ml.g5.2xlarge` | 1 NVIDIA A10G | Yes | Yes |
| `ml.g7e.2xlarge` | 1 NVIDIA RTX PRO 6000 | Yes | No |

Use only instance types supported by the selected Marketplace version. The notebook lists them as SageMaker instance pools, so an endpoint falls back to the next type when one lacks capacity. Not every type is offered in every AWS Region; the helper skips a type that SageMaker rejects in your Region and keeps the rest in order. The total context includes input, conversation history, tool definitions, generated reasoning and the final answer.

## Get started

1. Subscribe to the K2 Horizon product in AWS Marketplace and accept its terms. Copy the model package ARN for your deployment Region from the product’s deployment instructions. You create a SageMaker model from that package, then an endpoint configuration and an endpoint; the notebook does this for you.
2. Open the notebook in an Amazon SageMaker AI notebook environment with Python 3. Install the dependencies in the notebook’s kernel:

   ```python
   %pip install boto3 "sagemaker>=2.254,<3"
   ```

   Alternatively, clone this repository and install `requirements.txt` in your Python environment.

3. Set `PACKAGE_ARN` in the setup cell, or provide the `K2_MODEL_PACKAGE_ARN` environment variable. `INSTANCE_TYPES = None` uses every real-time type the package supports, in fallback order.
4. Run the cells in order. The notebook deploys an endpoint, runs the inference examples and cleans up the resources it creates. The batch section runs a separate temporary job; set `RUN_BATCH=False` to skip it.

The notebook uses the standard AWS credential chain and detects the SageMaker execution role. Outside SageMaker, configure AWS credentials and set `SAGEMAKER_EXECUTION_ROLE_ARN`. Never put access keys in the notebook.

Your AWS identity needs permission to create, inspect, invoke and delete SageMaker resources, and to pass the execution role. The execution role needs access to the model package and the example S3 input/output prefix. Deployment depends on regional instance availability and on your account's endpoint quota for each instance type ("<instance type> for endpoint usage" in Service Quotas), which can be zero in new accounts. Instance pools fall back when a type lacks capacity, not when your account has no quota for it, so check the quota for the types you deploy, starting with the first in your list.

**Batch host and quota:** for batch transform, set `TransformAmiVersion` to `al2-ami-sagemaker-batch-gpu-535` (the notebook does this); the default batch host can use an older GPU driver. Batch Transform has its own service quota, separate from endpoints, and new accounts often start at zero. Before your first batch job, request the "<instance type> for transform job usage" quota in Service Quotas; otherwise the job fails before the model starts.

## Costs and cleanup

The model has no software fee. AWS compute, storage and other infrastructure charges apply.

Keep the notebook running until cleanup confirms deletion. If execution is interrupted, retain the printed resource names and rerun `cleanup(session, resources)` or `cleanup_batch(session, batch_resources)`. Verify that the endpoint is deleted or the batch job is terminal before leaving. A local timeout does not stop AWS billing.

Setting `EXISTING_ENDPOINT` invokes an endpoint you own without deleting it. The batch example preserves your S3 bucket; object versions may remain if bucket versioning is enabled.

## Samples and model details

- [Real-time input](samples/k2-horizon-7b-fp8/realtime-input.json) and [actual output](samples/k2-horizon-7b-fp8/realtime-output.json).
- [Batch input](samples/k2-horizon-7b-fp8/batch-input.json) and [actual output](samples/k2-horizon-7b-fp8/batch-output.json).
- [Training, evaluation and deployment latency](docs/k2-horizon-7b-fp8.md), with [latency input](samples/k2-horizon-7b-fp8/latency-input.json) and [measurement](samples/k2-horizon-7b-fp8/latency-measurement.json).

## Model and support

- [Institute of Foundation Models](https://ifm.ai)
- [K2 Horizon 7B FP8 model card](https://huggingface.co/IFM/K2-Horizon-7B-FP8)
- Support: [support@ifm.ai](mailto:support@ifm.ai)

## License

The example code is licensed under [Apache License 2.0](LICENSE). Refer to each model’s model card and Marketplace terms for its license.
