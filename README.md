# Ensamblaje-de-lecturas-largas

## Convertir fast5 a fastq

```
guppy_basecaller -i fast5_pass --save_path fastq_hac --config /opt/ont/guppy/data/dna_r9.4.1_450bps_hac.cfg --compress_fastq --recursive --trim_barcodes --resume --cpu_threads_per_caller 16 --num_callers 1
```

## Descomprimir los archivos fastq.gz 

```
gunzip -d *.fastq.gz
cat *pass.fastq> all_records.fastq
```

## Estadísticas de las lecturas

```
NanoStat --fastq all_records.fastq > stat_all_records.txt
```

## Eliminar lecturas redundantes

```
seqkit rmdup -i all_records.fastq -o deduplicated.fastq
```

## Remover los cebadores o primers

```
porechop -i all_records.fastq -o porechop_records.fastq --threads 25
NanoStat --fastq porechop_records.fastq > stat_porechop_records.txt
```



