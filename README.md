# Ensamblaje-de-lecturas-largas

## Convertir fast5 a fastq

```
guppy_basecaller -i fast5_pass --save_path fastq_hac --config /opt/ont/guppy/data/dna_r9.4.1_450bps_hac.cfg --compress_fastq --recursive --trim_barcodes --resume --cpu_threads_per_caller 16 --num_callers 1
```

## Descomprimir los archivos fastq.gz 

```
gunzip -d *.fastq.gz
cat *pass.fastq> all_records.fastq
NanoStat --fastq all_records.fastq > stat_all_records.txt
```
