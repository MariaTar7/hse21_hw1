# hse21_hw1
 Создадим папку для работы.

    mkdir hw1
    cd hw1
 Создаем ссылки.
 
    ls /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}

С помощью команды seqtk выбираем случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs. Параметр равен 414.

    seqtk sample -s414 oil_R1.fastq 5000000 > pe_R1.fastq
    seqtk sample -s414 oil_R2.fastq 5000000 > pe_R2.fastq
    seqtk sample -s414 oilMP_S4_L001_R1_001.fastq 1500000 > mp_R1.fastq
    seqtk sample -s414 oilMP_S4_L001_R2_001.fastq 1500000 > mp_R2.fastq
Оцениваем качество исходных чтений и получаем по ним общую статистику.

    mkdir fastqc      
    ls *.fastq | xargs -P 4 -tI{} fastqc -o fastqc {}      
    mkdir multiqc      
    multiqc -o multiqc fastqc
С помощью программ platanus_trim и platanus_internal_trim подрезаем чтения.

    platanus_trim pe_R1.fastq pe_R2.fastq 
    platanus_internal_trim mp_R1.fastq mp_R2.fastq 
 С помощью программы fastQC и multiQC оцениваем качество подрезанных чтений и получаем по ним общую статистику.
 
    mkdir trimmed_fastqc
    mkdir trimmed_multiqc
    ls *.trimmed | xargs -tI{} fastqc -o trimmed_fastqc {}
    multiqc -o trimmed_multiqc trimmed_fastqc
Удаляем файлы.

    rm mp*.fastq           
    rm pe*.fastq
    
Собираем контиги из подрезанных чтений.

    time platanus assemble -o Poil -t 1 -m 8 -f *.trimmed 2>assemble.log
    
Собираем скаффолды из контигов, а также из подрезанных чтений.

    time platanus scaffold -o Poil -t 1 -c Poil_contig.fa -IP1 *.trimmed -OP2 *.int_trimmed 2>scaffold

С помощью программы “ platanus gap_close” уменьшаем кол-во гэпов с помощью подрезанных чтений.

    platanus gap_close -o Poil -t 1 -c Poil_scaffold.fa -IP1 *.trimmed -OP2 *.int_trimmed 2>gapclose.log
    
