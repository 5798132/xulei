# xulei
糖尿病人的口腔样本
cd-hit-est -i meta.fasta -o meta_derepprefix.cdhit.c0.9865.fasta -n 9 -c 0.9865 -G 0 -M 0 -d 0 -aS 0.9 -r 0 -T 40
less meta_derepprefix.cdhit.c0.9865.fasta.clstr | perl -e 'my $head = <>;chomp $head;$head =~ s/\>//;$head =~ s/\ //;print "$head";while(<>){chomp;if(/^\>Clust/){$_ =~ s/\>//;$_ =~ s/\ //;print "\n$_"}else{my @t = split/\,/;my @a = split/\.\.\./,$t[1];$a[0] =~ s/\>//;$a[0] =~ s/\ //;print "\t$a[0]"}};print "\n"' > otu_seqid.cdhit.c0.9865.txt
format_seqid.pl otu_seqid.cdhit.c0.9865.txt otu_seqid.cdhit
pick_rep_set.py -i otu_seqid.cdhit.txt -m most_abundant -f meta.fasta -o otu_rep.fasta
sample_nameCheck.pl otu_seqid.cdhit.txt otu_seqid F
make_otu_table.py -i otu_seqid.rename.txt -o otu_table.rename.biom
biom convert -i otu_table.rename.biom -o otu_table.rename.txt --table-type "OTU table" --to-tsv
format_otuTable_toXls.pl otu_table.rename.txt otu_table.rename.all.xls
filter_single.pl otu_table.rename.all.xls F otu_table.rename.xls
sample_nameBack.pl otu_table.rename.xls otu_seqid.namecheck otu_table.no-order.xls
sample_order.pl otu_table.no-order.xls trim.fq.list otu_table.xls
biom convert -i otu_table.xls -o otu_table.biom --table-type "OTU table" --to-json
mv otu_rep.fasta otu_rep.temp.fasta
getSeq_byName.pl otu_table.xls otu_rep.temp.fasta otu_rep.fasta
assign_taxonomy.py -m uclust -i otu_rep.fasta --similarity 0.8 -r silva.16s.ncbi-genus.fasta -t silva.16s.ncbi-genus.tax -o silva-uclust0.8
cp silva-uclust0.8/otu_rep_tax_assignments.txt  rep_tax_assignments.txt
taxaTable_byAss.pl otu_table.xls rep_tax_assignments.txt otu_taxa_table.xls
biom convert -i otu_taxa_table.xls -o otu_taxa_table.biom --process-obs-metadata taxonomy --table-type "OTU table" --to-json
