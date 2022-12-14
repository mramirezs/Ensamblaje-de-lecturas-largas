# Ensamblaje-de-lecturas-largas

## Instalar el driver de la tarjeta de video (NVIDIA)

```
sudo apt install nvidia-utils-525
```

## Instalación de Guppy

* Guppy se distribuye como un archivo tar y está disponible a través de su web comunitaria https://community.nanoporetech.com/. El archivo normalmente tiene un nombre como ont-guppy_version_linux64.tar.gz (versión GPU) y ont-guppy-cpu_version_linux64.tar.gz (versión CPU).

* Nosotros trabajaremos con la versión 6.3.7:

```
wget https://mirror.oxfordnanoportal.com/software/analysis/ont-guppy_6.3.7_linux64.tar.gz
tar xvfz ont-guppy_6.3.7_linux64.tar.gz 
cd ont-guppy/
cd bin/
./guppy_basecaller --help
pwd
sudo ln -s /home/prosopis/Bioprograms/ont-guppy/bin/guppy_basecaller /usr/local/bin/guppy_basecaller
```

## Convertir fast5 a fastq

Un requerimiento para el llamado de las bases en  es conocer el flowcell que se usó en el secuenciador, para nuestros caso tenemos FLO-MIN106, con el kit ADN genómico SQK-LSK110 y para ARN el kit SQK-PCB109.

Para conocer la listas de workflows usamos el comando:

```
$ guppy_basecaller --print_workflows | head
```

Output:

```
CRASHPAD MESSAGE: 
Loading model version information, please wait ..............................................................................................................
Available flowcell + kit combinations are:
flowcell       kit               barcoding config_name                    model version
FLO-MIN112     SQK-LSK112                  dna_r10.4_e8.1_hac             2021-09-03_dna_r10.4_minion_promethion_384_6b8e75c7
FLO-MIN112     SQK-LSK112-XL               dna_r10.4_e8.1_hac             2021-09-03_dna_r10.4_minion_promethion_384_6b8e75c7
FLO-MIN112     SQK-RAD112                  dna_r10.4_e8.1_hac             2021-09-03_dna_r10.4_minion_promethion_384_6b8e75c7
FLO-MIN112     SQK-NBD112-24     included  dna_r10.4_e8.1_hac             2021-09-03_dna_r10.4_minion_promethion_384_6b8e75c7
FLO-MIN112     SQK-NBD112-96     included  dna_r10.4_e8.1_hac             2021-09-03_dna_r10.4_minion_promethion_384_6b8e75c7
FLO-MIN112     SQK-RBK112-24     included  dna_r10.4_e8.1_hac             2021-09-03_dna_r10.4_minion_promethion_384_6b8e75c7
```
Para identificar nuestro flowcell usamos:

```
$ guppy_basecaller --print_workflows | grep "SQK-LSK110" | grep "FLO-MIN106"
```

Output:

```
FLO-MIN106     SQK-LSK110                  dna_r9.4.1_450bps_hac          2021-05-17_dna_r9.4.1_minion_384_d37a2ab9
FLO-MIN106     SQK-LSK110-XL               dna_r9.4.1_450bps_hac          2021-05-17_dna_r9.4.1_minion_384_d37a2ab9
```

Con los datos obtenidos se procese al llama:

```
guppy_basecaller -r -i fast5 -s fastq_Guppy_v6.3.7 --config dna_r9.4.1_450bps_sup.cfg  -q 0 --trim_strategy dna --disable_trim_barcodes --compress_fastq --calib_detect --device auto
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

## Filtrar las lecturas por longitud y por calidad de bases

```
NanoFilt -q 9 -l 9999 porechop_records.fastq > nanofilt_records.fastq
```

## Ensamblaje De Novo

```
flye --nano-raw nanofilt_records.fastq --out-dir flye --genome-size 426m --threads 30
```


