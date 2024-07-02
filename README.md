# AMXGPU

Performance optimized for 5.15.0-56-generic Kernel on SPR2

1. Run Docker Container with:  
`docker run --rm -it --gpus all -v /storage:/home/storage --privileged ipex-llm:flexgen bash`
2. Activate environment:  
`cd llm`  
`source ./tools/env_activate.sh`
3. Modify the following files:  
`~/miniconda3/envs/py310/lib/python3.10/site-packages/transformers/models/opt/modeling_opt.py`  
with AMXGPU/modeling_opt.py (in the current repository)  
`~/miniconda3/envs/py310/lib/python3.10/site-packages/intel_extension_for_pytorch/transformers/models/reference/modules/attentions.py`  
with IPEX_CUDA/intel_extension_for_pytorch/transformers/models/reference/modules/attentions.py (in the IPEX_CUDA repository)  
`~/miniconda3/envs/py310/lib/python3.10/site-packages/intel_extension_for_pytorch/transformers/models/reference/modules/decoder.py`  
with IPEX_CUDA/intel_extension_for_pytorch/transformers/models/reference/modules/decoder.py (in the IPEX_CUDA repository)  
`~/miniconda3/envs/py310/lib/python3.10/site-packages/intel_extension_for_pytorch/transformers/optimize.py`  
with IPEX_CUDA/intel_extension_for_pytorch/transformers/models/reference/optimize.py (in the IPEX_CUDA repository)  
`~/miniconda3/envs/py310/lib/python3.10/site-packages/intel_extension_for_pytorch/transformers/generation/greedy_search.py`  
with IPEX_CUDA/intel_extension_for_pytorch/transformers/models/reference/greedy_search.py (in the IPEX_CUDA repository)  
##
4. Run the following command:
`unset KMP_AFFINITY`
5. Run the command for opt-1.3b inference with deepspeed (opt-30b is also downloaded in /home/storage/hyungyo2/opt-model):  
`deepspeed --bind_cores_to_rank run.py --benchmark -m /home/storage/hyungyo2/opt-model/opt-1.3b/ --dtype bfloat16 --autotp --ipex --input-tokens 64 --max-new-tokens 32 --batch-size 1 --token-latency --num-iter 2 --num-warmup 1 --greedy`
##
Memory pinning is done in AMXGPU/modeling_opt.py line 128, 1135, 1145, 1243, and 1246
The code is currently very dirty. I just wanted to make the framework run and didn't bother to clean it up.

When NUMA node 2, 3 are configured for the CXL expander in SPR2, the performance drops significantly. I think it's because it assigns some data to CXL memory.
Let's figure this problem out.
