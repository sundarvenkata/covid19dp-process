# Covid-19 data ingestion process 
#### (click on nodes to navigate to the relevant code/documentation)
### The steps below are glued together using a [Nextflow pipeline](https://github.com/EBIvariation/covid19dp-submission/blob/3a6f3615bb7697a7fb2b51da79ba8d85d02b84b5/covid19dp_submission/nextflow/submission_workflow.nf)

```mermaid
graph TD
    user["Lab submits Covid19 variant data - patients and control group (~200GB per day)"] -->|via submission API|validateVCF{"<u>Syntax validation (parallel)</u>"}
	validateVCF --> |pass| validateSemantics{"<u>Semantic validation (parallel)</u>"}
	validateVCF --> |fail - email submitter| user
	validateSemantics --> |fail - email submitter| user
	validateSemantics --> |pass| semanticMergeGenerate(<u>Generate dynamic multi-stage merge pipeline for data files</u>)
	semanticMergeGenerate --> semanticMerge("<u>Run multi-stage merge in HPC cluster (20-30k files in parallel per day)</u>")
	semanticMerge --> accession("<u>Provide unique identifiers for the submitted genomic variants</u>")
	accession --> clustering("<u>Cluster common genomic variants observed across multiple subjects</u>")
	clustering --> incrementalRelease[("<u>Incremental release records in Mongo for auditing</u>")]
	incrementalRelease --> kafkaPublish[("<u>Publish to an enterprise Kafka topic for use by other teams</u>")]
	kafkaPublish --> |subscribed by| proteinCoding("<u>Protein coding</u>") 
	kafkaPublish --> |subscribed by| dataPortal("<u>Covid-19 data portal</u>")
	kafkaPublish --> |subscribed by| enterpriseSearch("<u>Enterprise search</u>")
	
	click validateVCF "https://github.com/EBIvariation/vcf-validator"
	click validateSemantics "https://github.com/EBIvariation/vcf-validator/blob/master/src/assembly_checker_main.cpp"
	click semanticMergeGenerate "https://github.com/EBIvariation/covid19dp-submission/blob/3a6f3615bb7697a7fb2b51da79ba8d85d02b84b5/covid19dp_submission/steps/vcf_vertical_concat/run_vcf_vertical_concat_pipeline.py#L41"
	click semanticMerge "https://github.com/EBIvariation/covid19dp-submission/blob/3a6f3615bb7697a7fb2b51da79ba8d85d02b84b5/covid19dp_submission/nextflow/submission_workflow.nf#L67"
	click accession "https://github.com/EBIvariation/eva-accession/"
	click clustering "https://github.com/EBIvariation/eva-accession/blob/f9863197a232b535a6b2b8915440f1c4059e48dd/eva-accession-clustering/src/main/java/uk/ac/ebi/eva/accession/clustering/configuration/batch/jobs/ClusteringFromMongoJobConfiguration.java#L98"
	click incrementalRelease "https://github.com/EBIvariation/eva-accession/blob/f9863197a232b535a6b2b8915440f1c4059e48dd/eva-accession-release/src/main/java/uk/ac/ebi/eva/accession/release/configuration/batch/steps/CreateIncrementalReleaseStepConfiguration.java#L87"
	click proteinCoding "https://www.covid19dataportal.org/search/proteins?db=uniprot-covid19&size=15&facets=TAXONOMY:2697049&crossReferencesOption=all"
	click dataPortal "https://www.covid19dataportal.org/search/sequences?crossReferencesOption=all&overrideDefaultDomain=true&db=eva-variants-covid19&size=15"
  ```
