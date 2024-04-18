# hello-nf-test


Now that you have a functioning workflow let's discuss about making this pipeline more modular usig Nextflow's DSL2 & adding unit-tests to your pipeline.


```bash
mkdir -p modules/local/samtools/index/tests
mkdir -p modules/local/gatk/haplotypecaller/tests
mkdir -p modules/local/gatk/jointgenotyping/tests
```


```bash
touch modules/local/samtools/index/main.nf
touch modules/local/gatk/haplotypecaller/main.nf
touch modules/local/gatk/jointgenotyping/main.nf
```

```

* Copy each process block into their respective `main.nf` files.

* Modify the `hello-gatk.nf` workflow to now have `include {}` statements for the three modules

```bash
// Include modules

include { SAMTOOLS_INDEX } from './modules/local/samtools/index/main.nf'
include { GATK_HAPLOTYPECALLER } from './modules/local/gatk/haplotypecaller/main.nf'
include { GATK_JOINTGENOTYPING } from './modules/local/gatk/jointgenotyping/main.nf'
```

* Should work the same now...give it a try!

---

* Next lets add some tests

* nf-test init

```bash
nf-test init
```

```bash
🚀 nf-test 0.8.4
https://code.askimed.com/nf-test
(c) 2021 - 2024 Lukas Forer and Sebastian Schoenherr

Project configured. Configuration is stored in nf-test.config
```

* nf-test generate process test files for modules

```bash
nf-test generate process modules/local/samtools/index/main.nf
nf-test generate process modules/local/gatk/haplotypecaller/main.nf
nf-test generate process modules/local/gatk/jointgenotyping/main.nf
```

* Good practice to ship tests along with each `main.nf` so, let's move the files generated to their respective `tests/` folders

```bash
mv /workspace/gitpod/hello-nextflow/tests/modules/local/samtools/index/main.nf.test modules/local/samtools/index/tests/
mv /workspace/gitpod/hello-nextflow/tests/modules/local/gatk/haplotypecaller/main.nf.test modules/local/gatk/haplotypecaller/tests/
mv /workspace/gitpod/hello-nextflow/tests/modules/local/gatk/jointgenotyping/main.nf.test modules/local/gatk/jointgenotyping/tests/
```

* Let's work on samtools index first. Start by replacing the absolute path in the `script` section to relative path now

```groovy
nextflow_process {

    name "Test Process SAMTOOLS_INDEX"
    script "../main.nf"
    process "SAMTOOLS_INDEX"

    test("reads_son [bam]") {

        when {
            params {
                outdir = "tests/results"
            }
            process {
                """
                input[0] = [ [id: 'NA12882' ], file("/workspace/gitpod/hello-nextflow/data/bam/reads_son.bam") ]
                """
            }
        }

        then {
            assert process.success
            assert snapshot(process.out).match()
        }

    }

}
```

* run the test

```bash
nf-test test modules/local/samtools/index/tests/main.nf.test
```


```bash
🚀 nf-test 0.8.4
https://code.askimed.com/nf-test
(c) 2021 - 2024 Lukas Forer and Sebastian Schoenherr


Test Process SAMTOOLS_INDEX

  Test [bc664c47] 'reads_son [bam]' PASSED (9.087s)
  Snapshots:
    1 created [reads_son [bam]]


Snapshot Summary:
  1 created

SUCCESS: Executed 1 tests in 9.095s
```

* This will create a snapshot `main.nf.test` of all the output channels, which in this case is the bam file produced containing the MD5SUM

```groovy
{
    "reads_son [bam]": {
        "content": [
            {
                "0": [
                    [
                        {
                            "id": "NA12882"
                        },
                        "reads_son.bam:md5,12e4f6133168de9441cd679347ed249b",
                        "reads_son.bam.bai:md5,ac1eb007b3c187783b0f3a70044930b9"
                    ]
                ]
            }
        ],
        "meta": {
            "nf-test": "0.8.4",
            "nextflow": "24.02.0"
        },
        "timestamp": "2024-04-18T20:03:35.18709459"
    }
}
```

* Since there are three inputs, and unit-testing entails testing for all inputs let's add the tests for mother and father input bams as well

```groovy
nextflow_process {

    name "Test Process SAMTOOLS_INDEX"
    script "../main.nf"
    process "SAMTOOLS_INDEX"

    test("reads_son [bam]") {

        when {
            params {
                outdir = "tests/results"
            }
            process {
                """
                input[0] = [ [id: 'NA12882' ], file("/workspace/gitpod/hello-nextflow/data/bam/reads_son.bam") ]
                """
            }
        }

        then {
            assert process.success
            assert snapshot(process.out).match()
        }

    }

    test("reads_mother [bam]") {

        when {
            params {
                outdir = "tests/results"
            }
            process {
                """
                input[0] = [ [id: 'NA12878' ], file("/workspace/gitpod/hello-nextflow/data/bam/reads_mother.bam") ]
                """
            }
        }

        then {
            assert process.success
            assert snapshot(process.out).match()
        }

    }

    test("reads_father [bam]") {

        when {
            params {
                outdir = "tests/results"
            }
            process {
                """
                input[0] = [ [id: 'NA12877' ], file("/workspace/gitpod/hello-nextflow/data/bam/reads_father.bam") ]
                """
            }
        }

        then {
            assert process.success
            assert snapshot(process.out).match()
        }

    }

}
```

* Now let's re-run with mother and father tests included in the same file and use the parameter `--update-snpashot` to add those entries to the snapshot

```bash
nf-test test modules/local/samtools/index/tests/main.nf.test --update-snapshot
```

```bash
🚀 nf-test 0.8.4
https://code.askimed.com/nf-test
(c) 2021 - 2024 Lukas Forer and Sebastian Schoenherr

Warning: every snapshot that fails during this test run is re-record.

Test Process SAMTOOLS_INDEX

  Test [bc664c47] 'reads_son [bam]' PASSED (12.001s)
  Test [f413ec92] 'reads_mother [bam]' PASSED (10.384s)
  Test [99a73481] 'reads_father [bam]' PASSED (9.543s)
  Snapshots:
    2 created [reads_father [bam], reads_mother [bam]]


Snapshot Summary:
  2 created
```

```groovy
{
    "reads_father [bam]": {
        "content": [
            {
                "0": [
                    [
                        {
                            "id": "NA12877"
                        },
                        "reads_father.bam:md5,961b7b7c7d8945ff0d93861d18610a83",
                        "reads_father.bam.bai:md5,0415dc1de5f798f398322fa84a1bd403"
                    ]
                ]
            }
        ],
        "meta": {
            "nf-test": "0.8.4",
            "nextflow": "24.02.0"
        },
        "timestamp": "2024-04-18T20:09:09.800402231"
    },
    "reads_son [bam]": {
        "content": [
            {
                "0": [
                    [
                        {
                            "id": "NA12882"
                        },
                        "reads_son.bam:md5,12e4f6133168de9441cd679347ed249b",
                        "reads_son.bam.bai:md5,ac1eb007b3c187783b0f3a70044930b9"
                    ]
                ]
            }
        ],
        "meta": {
            "nf-test": "0.8.4",
            "nextflow": "24.02.0"
        },
        "timestamp": "2024-04-18T20:03:35.18709459"
    },
    "reads_mother [bam]": {
        "content": [
            {
                "0": [
                    [
                        {
                            "id": "NA12878"
                        },
                        "reads_mother.bam:md5,e6c1fc44bd6831a8b0c532c5d824931c",
                        "reads_mother.bam.bai:md5,e67e4b0277dddeacb0a5016e41f19832"
                    ]
                ]
            }
        ],
        "meta": {
            "nf-test": "0.8.4",
            "nextflow": "24.02.0"
        },
        "timestamp": "2024-04-18T20:09:00.269007074"
    }
}
```