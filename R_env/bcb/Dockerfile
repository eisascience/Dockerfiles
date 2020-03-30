from bioconductor/bioconductor_docker:RELEASE_3_10

RUN apt-get update -y \
	&& apt-get install -y \
		libhdf5-dev \
		libpython3-dev \
		python3-pip \
		openjdk-11-jdk \
    && pip3 install umap-learn \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/*

# Let this run for the purpose of installing/caching dependencies
RUN Rscript -e "install.packages(c('devtools', 'BiocManager', 'remotes'), dependencies=TRUE, ask = FALSE)" \
    && echo -e "local({\noptions(repos = BiocManager::repositories())\n})\n" >> ~/.Rprofile.site \
    # NOTE: these seem to be required for garnett to succeed in docker. DESeq2/genefilter added for Seurat
    && Rscript -e "BiocManager::install(c('org.Hs.eg.db', 'org.Mm.eg.db', 'HSMMSingleCell', 'monocle', 'DelayedMatrixStats', 'DESeq2', 'genefilter'), dependencies=TRUE, ask = FALSE)" \
    && Rscript -e "BiocManager::install(c('ggplot2', 'Matrix', 'plotly', 'shiny', 'shinyWidgets', 'rclipboard', 'shinydashboard', 'data.table', 'ggrepel', 'viridis', 'RColorBrewer', 'grid', 'gridExtra', 'dplyr', 'shinyFiles', 'BiocParallel', 'ggnewscale', 'ggpubr'), dependencies=TRUE, ask = FALSE)" \
    && Rscript -e "BiocManager::install(c('flexdashboard', 'shinyMatrix', 'shinyBS', 'rsconnect', 'ggupset', 'tidyr', 'reshape2', 'stringr', 'readxl', 'datapasta', 'cowplot', 'kableExtra', 'DT', 'formattable', 'rhandsontable', 'stats', 'testthat', 'AnnotationHub', 'ReactomePA', 'clusterProfiler', 'DOSE', 'enrichplot', 'fgsea', 'biomaRt', 'STRINGdb', 'RDAVIDWebService', 'msigdbr'), dependencies=TRUE, ask = FALSE)" \
    && Rscript -e "devtools::install_github(repo = 'bimberlabinternal/OOSAP', ref = 'Dev', dependencies = T, upgrade = 'always')" \
    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds

# This should not be cached if the files change
ADD . /OOSAP

RUN cd /OOSAP \
    && R CMD build . \
    && Rscript -e "print(getOption('repos'))" \
    && Rscript -e "BiocManager::install(ask = F)" \
    && Rscript -e "devtools::install_deps(pkg = '.', dependencies = TRUE, threads = getOption('Ncpus',1))" \
    && R CMD INSTALL --build *.tar.gz \
    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds
