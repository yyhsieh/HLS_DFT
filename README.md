# HLS_DFT 
HLS program for DFT
Date: 2020/12/22

Reference: https://github.com/KastnerRG/pp4fpgas

## Project Description
This project utilizes discrete Fourier transform (DFT) to transfer the signal from time domain to frequency domain. And the primary method is to separate real and imaginary part, and use sine / cosine function to perform signal tranform. However the two-layer loop is the main bottleneck of rntire design, so I am wondering to optimize this two-layer loop for lower latency and better performance.

## HLS Optimization
original file: dft.c, dft_precompute.c, dft_unroll_inner2.c
the modified file by my own is ```dft_pipeline.c```

in the file dft_pipeline.c
```c= 15
	// Calculate each frequency domain sample iteratively
	for (i = 0; i < N; i += 1) {
		#pragma HLS pipeline II=32
		temp_real[i] = 0;
		temp_imag[i] = 0;

		// (2 * pi * i)/N
		w = (2.0 * 3.141592653589  / N) * (TEMP_TYPE)i;

		// Calculate the jth frequency sample sequentially
		for (j = 0; j < N; j += 1) {
			// Utilize HLS tool to calculate sine and cosine values
			c = cos(j * w);
			s = -sin(j * w);

			// Multiply the current phasor with the appropriate input sample and keep
			// running sum
			temp_real[i] += (sample_real[j] * c - sample_imag[j] * s);
			temp_imag[i] += (sample_real[j] * s + sample_imag[j] * c);
		}
	}

	// Perform an inplace DFT, i.e., copy result into the input arrays
	for (i = 0; i < N; i += 1) {
		#pragma HLS pipeline II=32
		sample_real[i] = temp_real[i];
		sample_imag[i] = temp_imag[i];
	}
}
```