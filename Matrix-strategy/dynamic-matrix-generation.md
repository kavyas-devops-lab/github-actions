 # Dynamic matrix strategy — fromJson
   The matrix strategies documented in GitHub is hard-coded/static entry.

  However, in some cases, environments change dynamically. Therefore, it would be beneficial if the matrix strategy is also dynamic when external environments change. In the above example, it would be good if we could discover all the node versions used in the environment and generate the matrix strategies accordingly so we don’t need to come back and update the code.

  # fromJson
  Here is the tree structure of the infrastructure folder, which holds the configuration files for all environments. I am interested in finding out all the folder names; however, I don’t need the ‘network-hub’ as this is NOT an environment.

```
infrastructure/
├── dev
├── network-hub
├── prod
├── qa
└── test
```

I created two GitHub Actions jobs:

## job: discover-environments:
It will loop through the entire ‘infrastructure’ folder and return all the environment folders, such as prod, dev, and qa, excluding the ‘network-hub’ folder.

```
discover-environments:
    name: Find out all the environments this customer have
    runs-on: ubuntu-latest
    outputs:
      environments: ${{ steps.list-environments.outputs.environments }}

    steps:
      - name: output all the environmnets in the infrastructure folder
        id: list-environments
        working-directory: infrastructure
        run: |
          directories=($(ls -d */))
          directories=("${directories[@]%/}")
          excluded_directories=("network-hub")
          json_array=()
          for dir in "${directories[@]}"; do
            if [[ ! " ${excluded_directories[@]} " =~ " $dir " ]]; then
              json_array+=("\"$dir\"")
            fi
          done
          json_elements=$(IFS=,; echo "${json_array[*]}")
          json_output="{ \"environments\": [$json_elements] }"
          echo "environments=$json_output" | tee -a "$GITHUB_OUTPUT"
```
It will generated json output like this:

```
environments={ "environments": ["dev","prod","qa","test"] }
```
## job: exploring
This exploration job uses matrix strategies to access each environment and obtain EKS deployments information from each environment.

```
 exploring:
    runs-on: ubuntu-latest
    if: always()
    needs: discover-environments
    strategy:
      matrix: ${{ fromJson(needs.discover-environments.outputs.environments) }}
```
This step runs the ‘kubectl get deployment’ command on each environment and uploads the output as artifacts.

```
 - name: Generate deployment output for environment ${{ matrix.environments }}
        run: |
          kubectl_output=$(kubectl get deploy -A -o custom-columns=namespace:.metadata.namespace,name:.metadata.name --no-headers | grep "deployment" || true)
          if [ -n "$kubectl_output" ]; then
            while read -r line; do
              IFS=" " read -r namespace deployment_name <<< "$line"
              echo "          - ${{ matrix.environments }} | $namespace | $deployment_name" >> ${{ matrix.environments }}-eks-deployments.txt
            done <<< "$kubectl_output"
            cat ${{ matrix.environments }}-eks-deployments.txt
          else
            echo "kubectl get deploy -A | grep deployment - return no result!!! That means no application deployment in ${{ matrix.environments }} environmnt"
          fi

      - name: Upload artifacts related to eks deployment in ${{ matrix.environments }}
        uses: actions/upload-artifact@v3
        with:
          name: eks-deployments
          path: ${{ env.TOOLS_BASE }}/${{ matrix.environments }}-eks-deployments.txt
```
output of test environment:
```
          - test | private | helloworld-1-deployment
          - test | private | helloworld-2-deployment
          - test | private | helloworld-3-deployment
          - test | private | helloworld-4-deployment
```
As matrices run simultaneously for each environment (different from a for-loop), we can later download all the artifacts from all the environments and create a list.

```
- name: Download all workflow run artifacts
        uses: actions/download-artifact@v3

```

