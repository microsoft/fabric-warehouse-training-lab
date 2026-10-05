# fabric-warehouse-training-lab
The Fabric Warehouse training lab jump start

## Deployment parameterization

The root `parameter.yml` is loaded automatically by Fabric Jumpstart's
`fabric-cicd` deployment. For every target environment, it replaces the source
workspace GUID with `$workspace.$id` and the source lakehouse GUID with the
deployed ID of `warehouse_training_lab_sample_data`.

These replacements cover all notebook OneLake paths, including `COPY INTO` and
`OPENROWSET`, and the data generator's lakehouse attachment. Deploy the Lakehouse
alongside the notebooks and keep `parameter.yml` at the deployment repository
root. Run `generate tpc-h data` before the T-SQL labs.

Direct notebook import or Fabric Git synchronization does not apply this
`fabric-cicd` parameter file; those workflows require separate parameterization.
