# Covid-19 data ingestion process 
#### (click on nodes to navigate to the relevant code)

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
	clustering --> incrementalRelease[("<u>Incremental release records for auditing</u>")]
	click validateVCF "https://github.com/EBIvariation/covid19dp-submission/blob/3a6f3615bb7697a7fb2b51da79ba8d85d02b84b5/covid19dp_submission/nextflow/submission_workflow.nf#L1"
	click validateSemantics "https://github.com/EBIvariation/covid19dp-submission/blob/3a6f3615bb7697a7fb2b51da79ba8d85d02b84b5/covid19dp_submission/nextflow/submission_workflow.nf#L22"
	click semanticMergeGenerate "https://github.com/EBIvariation/covid19dp-submission/blob/3a6f3615bb7697a7fb2b51da79ba8d85d02b84b5/covid19dp_submission/steps/vcf_vertical_concat/run_vcf_vertical_concat_pipeline.py#L41"
	click semanticMerge "https://github.com/EBIvariation/covid19dp-submission/blob/3a6f3615bb7697a7fb2b51da79ba8d85d02b84b5/covid19dp_submission/nextflow/submission_workflow.nf#L67"
	click accession "https://github.com/EBIvariation/eva-accession/"
	click clustering "https://github.com/EBIvariation/eva-accession/blob/f9863197a232b535a6b2b8915440f1c4059e48dd/eva-accession-clustering/src/main/java/uk/ac/ebi/eva/accession/clustering/configuration/batch/jobs/ClusteringFromMongoJobConfiguration.java#L98"
	click incrementalRelease "https://github.com/EBIvariation/covid19dp-submission/blob/3a6f3615bb7697a7fb2b51da79ba8d85d02b84b5/covid19dp_submission/nextflow/submission_workflow.nf#L153"
  ```
