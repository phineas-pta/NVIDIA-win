# Test 9: build Stan with OpenCL

📑 in case u need official docs:
- https://mc-stan.org/docs/cmdstan-guide/cmdstan-installation.html
- https://github.com/bstatcomp/stan_gpu_install_docs/blob/master/install_docs/INSTALL_CMDSTAN.md
- https://blog.mc-stan.org/2022/08/03/options-for-improving-stan-sampling-speed/

prepare at least 2 GiB disk space

tested version: CmdStan v2.35.0: https://github.com/stan-dev/cmdstan/releases/download/v2.35.0/cmdstan-2.35.0.tar.gz

untar the file then create a file named `local` inside the `make` folder with:
```
STAN_CPP_OPTIMS=true
CXXFLAGS+=-mtune=native -march=native
CFLAGS+=-mtune=native -march=native
CXXFLAGS_OPTIM_SUNDIALS+=-mtune=native -march=native
CPPFLAGS_OPTIM_SUNDIALS+=-mtune=native -march=native
CXXFLAGS_OPTIM_TBB+=-mtune=native -march=native
STAN_THREADS=true
STAN_OPENCL=true
OPENCL_PLATFORM_ID=0
OPENCL_DEVICE_ID=0
LDFLAGS_OPENCL=-L"C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.6/lib/x64" -lOpenCL
```

need Rtools Msys2 console setup with `mingw32-make` + `g++`
```
mingw32-make -j build
```

additional step: `export PATH=$PATH:███/cmdstan-2.35.0/stan/lib/stan_math/lib/tbb`

**R script for installation**
```r
install.packages("cmdstanr", repos = c("https://stan-dev.r-universe.dev", getOption("repos")))
library(cmdstanr)

check_cmdstan_toolchain()
set_cmdstan_path("<PATH_TO_INSTALL_CMDSTAN>")

cmdstan_make_local(cpp_options = list(
	"STAN_CPP_OPTIMS" = TRUE,
	"CXXFLAGS+=-mtune=native -march=native",
	"CFLAGS+=-mtune=native -march=native",
	"CXXFLAGS_OPTIM_SUNDIALS+=-mtune=native -march=native",
	"CPPFLAGS_OPTIM_SUNDIALS+=-mtune=native -march=native",
	"CXXFLAGS_OPTIM_TBB+=-mtune=native -march=native",
	"STAN_THREADS" = TRUE,
	"STAN_OPENCL" = TRUE,
	"OPENCL_PLATFORM_ID" = 0,
	"OPENCL_DEVICE_ID" = 0,
	"LDFLAGS_OPENCL" = sprintf('-L"%s/lib/x64" -lOpenCL', gsub("\\", "/", Sys.getenv("CUDA_PATH"), fixed = TRUE))
))

install_cmdstan(cores = parallel::detectCores(logical = FALSE), overwrite = TRUE)
```

**run test**

get my test files and put in the `examples` folder:
- stan file: https://github.com/phineas-pta/Bayesian-Methods-for-Hackers-using-PyStan/blob/main/stan%20files/test0ex1_noOutliers_full.stan
- data file: https://github.com/phineas-pta/Bayesian-Methods-for-Hackers-using-PyStan/blob/main/stan%20files/test0ex1_noOutliers.data.json

test:
```
mingw32-make -j examples/test0ex1_noOutliers_full.exe

examples/test0ex1_noOutliers_full.exe optimize data file=examples/test0ex1_noOutliers.data.json output file=examples/test0ex1_optim.csv

examples/test0ex1_noOutliers_full.exe sample num_chains=4 num_samples=50000 num_warmup=10000 thin=5 data file=examples/test0ex1_noOutliers.data.json output file=examples/test0ex1_fit.csv diagnostic_file=examples/test0ex1_dia.csv refresh=5000 num_threads=4

bin/stansummary.exe examples/test0ex1_fit_*.csv

examples/test0ex1_noOutliers_full.exe variational data file=examples/test0ex1_noOutliers.data.json output file=examples/test0ex1_advi.csv

examples/test0ex1_noOutliers_full.exe diagnose data file=examples/test0ex1_noOutliers.data.json output file=examples/test0ex1_diag.csv
```
