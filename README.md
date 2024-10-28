# Adaptyv Bio Competition Metrics

In round 2 of our [protein design competition](https://design.adaptyvbio.com/) we are using a rank average of three metrics: iPTM, ESM PLL and PAE interaction. Here you can find some details on how to compute those metrics for your proteins.

## Installation

See [here](https://github.com/sokrypton/ColabFold?tab=readme-ov-file) for instructions on how to install the ColabFold pipeline. We use it to generate structure predictions.

To run the python scripts in this repository, you will need to install the requirements with pip.

```bash
python -m pip install -r requirements.txt
```

## Running AlphaFold2

iPTM and PAE interaction are both derived from the output of AlphaFold2. More specifically, in our pipeline we use the ColabFold implementation.

To generate structure predictions with the same parameters as we do, you can use the following command.

```bash
colabfold_batch input.fasta output_dir --num-recycle 3 --num-seeds 3 --num-models 5 --templates
```

Here `input.fasta` is the input file with the protein sequences and `output_dir` is the directory where the predictions will be saved. Down the line we use the output that is ranked the highest by the ColabFold pipeline.

In the fasta file, each entry should have the binder and the target sequences, separated by `:`. In our processing script we assume that the binder sequence always comes first.

This command will use the ColabFold server to run MSA predictions. In case you are evaluating a large number of proteins, it is recommended to set up your own MSA pipeline, as described [here](https://github.com/YoshitakaMo/localcolabfold). If you do that, you will need to run those two commands (the former is run on your MSA server and the latter on a machine that has a GPU).

```bash
colabfold_search input.fasta database msa_dir --db2 pdb100_230517 --use-templates 1
colabfold_batch {NAME}.a3m output_dir --num-recycle 3 --num-seeds 3 --num-models 5 --templates --local-pdb-path {MOUNT_PATH}/20240101/pub/pdb/data/structures/divided/mmCIF --pdb-hit-file {NAME}_pdb100_230517.m8
```

Here `{NAME}.a3m` and `{NAME}_pdb100_230517.m8` are the MSA and template files generated by `colabfold_search` for a single sequence and `msa_dir` and `output_dir` are the directories where the outputs will be saved. Finally, `{MOUNT_PATH}` is the path to the PDB database (`s3://pdbsnapshots/`). The database can be mounted, for instance, with [`s3fs`](https://github.com/s3fs-fuse/s3fs-fuse).

## Extracting AlphaFold2 metrics

You can find a script for extracting iPTM and PAE interaction from the AlphaFold2 predictions with our python script. Note that this script assumes that the binder sequence always comes before the target sequence in the input fasta file.

```bash
python extract_metrics.py output_dir {NAME} --target_length {TARGET_LENGTH}
```

Here `{NAME}` is the name of the protein. Most of the time it is the name you provided in the input fasta file. The `output_dir` parameter should point to a folder generated with commands from the previous section. The script will output the iPTM and PAE interaction metrics for the protein. The `--target_length` flag is optional and should be used if the target sequence is not EGFR (as in our competition).

## Computing ESM PLL

The ESM PLL metric can be computed with a separate script. This assumes that your machine has a GPU.

```bash
python compute_esm_pll.py {BINDER_SEQUENCE}
```

Here `{BINDER_SEQUENCE}` is the amino acid sequence of the binder protein. The script will output the ESM PLL metric for this sequence.

## Rank average

To compute the final value, we rank the designs on each of the three metrics separately and then average the ranks. Ties are resolved by averaging the ranks of the tied submissions. The lower the rank average, the better the submission.