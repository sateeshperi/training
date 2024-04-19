# hello-nf-test


## Switch to DSL2

* Now that you have a functioning workflow let's explore further about making this pipeline more modular using Nextflow's DSL2 & adding unit-tests to your pipeline using nf-test.


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

---

* Copy each process block into their respective `main.nf` files.

* Modify the `hello-gatk.nf` workflow to now have `include {}` statements for the three modules

```bash
// Include modules

include { SAMTOOLS_INDEX } from './modules/local/samtools/index/main.nf'
include { GATK_HAPLOTYPECALLER } from './modules/local/gatk/haplotypecaller/main.nf'
include { GATK_JOINTGENOTYPING } from './modules/local/gatk/jointgenotyping/main.nf'
```

* Should work just the same as before now...give it a try!

----------------------------------------------------------------------------------------------------------------------

## Module nf-tests

* Next lets add some tests. Read blogpost on [nf-test in nf-core](https://nextflow.io/blog/2024/nf-test-in-nf-core.html)

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

* Let's work on `samtools/index` first. Start by replacing the absolute path in the `script` section to relative path `../main.nf` now and provide inputs in the `then` block with positional inputs `input[0]`.

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

* This will create a snapshot `main.nf.test.snap` capturing all the output channels and the MD5SUM's of all elements

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

* Now let's re-run with mother and father tests included in the same file and use the parameter `--update-snapshot` to add those entries to the snapshot

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

--------------------------------------------------------------------------------------------------------------------------------------------
## GATK_HAPLOTYPECALLER Module Tests
--------------------------------------------------------------------------------------------------------------------------------------------

* `GATK_HAPLOTYPECALLER` module is an example of what we called as a chained module i.e., it requires the output of another module as one of it's inputs. nf-test offers a [setup method](https://www.nf-test.com/docs/testcases/setup/) to facilitate this

* Update the `main.nf` of `modules/local/gatk/haplotypecaller` to below

```groovy
nextflow_process {

    name "Test Process GATK_HAPLOTYPECALLER"
    script "../main.nf"
    process "GATK_HAPLOTYPECALLER"

    test("reads_son [bam]") {

        setup {
            run("SAMTOOLS_INDEX") {
                script "../../../samtools/index/main.nf"
                process {
                    """
                    input[0] =  [ [id: 'NA12882' ], file("/workspace/gitpod/hello-nextflow/data/bam/reads_son.bam") ]
                    """
                }
            }
        }

        when {
            params {
                outdir = "tests/results"
            }
            process {
                """
                input[0] = SAMTOOLS_INDEX.out
                input[1] = file("${baseDir}/data/ref/ref.fasta")
                input[2] = file("${baseDir}/data/ref/ref.fasta.fai")
                input[3] = file("${baseDir}/data/ref/ref.dict")
                input[4] = file("${baseDir}/data/intervals.list")
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

* Run test

```bash
nf-test test modules/local/gatk/haplotypecaller/tests/main.nf.test
```

```bash
🚀 nf-test 0.8.4
https://code.askimed.com/nf-test
(c) 2021 - 2024 Lukas Forer and Sebastian Schoenherr


Test Process GATK_HAPLOTYPECALLER

  Test [86fd1bce] 'reads_son [bam]' PASSED (19.85s)
  Snapshots:
    1 created [reads_son [bam]]


Snapshot Summary:
  1 created

SUCCESS: Executed 1 tests in 19.856s
```

* Now if you run the test again, you will see that the test fails with differing md5sum on the vcf file. This is expected currently for vcf/bam files due to their non-deterministic nature

* Instead we will employ [other ways of output assertions](https://nf-co.re/docs/contributing/tutorials/nf-test_assertions), in this case - read the lines of vcf file and check for existence of specific lines in the vcf file

* Now lets run tests for all three inputs

```groovy
nextflow_process {

    name "Test Process GATK_HAPLOTYPECALLER"
    script "../main.nf"
    process "GATK_HAPLOTYPECALLER"

    test("reads_son [bam]") {

        setup {
            run("SAMTOOLS_INDEX") {
                script "../../../samtools/index/main.nf"
                process {
                    """
                    input[0] =  [ [id: 'NA12882' ], file("/workspace/gitpod/hello-nextflow/data/bam/reads_son.bam") ]
                    """
                }
            }
        }

        when {
            params {
                outdir = "tests/results"
            }
            process {
                """
                input[0] = SAMTOOLS_INDEX.out
                input[1] = file("${baseDir}/data/ref/ref.fasta")
                input[2] = file("${baseDir}/data/ref/ref.fasta.fai")
                input[3] = file("${baseDir}/data/ref/ref.dict")
                input[4] = file("${baseDir}/data/intervals.list")
                """
            }
        }

        then {
            assert process.success
            assert path(process.out[0][0][1]).readLines().contains('#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	NA12882')
            assert path(process.out[0][0][1]).readLines().contains('20	10040001	.	T	<NON_REF>	.	.	END=10040048	GT:DP:GQ:MIN_DP:PL	0/0:40:99:37:0,99,1150')
        }
    }

    test("reads_mother [bam]") {

        setup {
            run("SAMTOOLS_INDEX") {
                script "../../../samtools/index/main.nf"
                process {
                    """
                    input[0] =  [ [id: 'NA12882' ], file("/workspace/gitpod/hello-nextflow/data/bam/reads_mother.bam") ]
                    """
                }
            }
        }

        when {
            params {
                outdir = "tests/results"
            }
            process {
                """
                input[0] = SAMTOOLS_INDEX.out
                input[1] = file("${baseDir}/data/ref/ref.fasta")
                input[2] = file("${baseDir}/data/ref/ref.fasta.fai")
                input[3] = file("${baseDir}/data/ref/ref.dict")
                input[4] = file("${baseDir}/data/intervals.list")
                """
            }
        }

        then {
            assert process.success
            assert path(process.out[0][0][1]).readLines().contains('#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	NA12878')
            assert path(process.out[0][0][1]).readLines().contains('20	10040001	.	T	<NON_REF>	.	.	END=10040013	GT:DP:GQ:MIN_DP:PL	0/0:28:81:27:0,81,829')
        }
    }

    test("reads_father [bam]") {

        setup {
            run("SAMTOOLS_INDEX") {
                script "../../../samtools/index/main.nf"
                process {
                    """
                    input[0] =  [ [id: 'NA12882' ], file("/workspace/gitpod/hello-nextflow/data/bam/reads_father.bam") ]
                    """
                }
            }
        }

        when {
            params {
                outdir = "tests/results"
            }
            process {
                """
                input[0] = SAMTOOLS_INDEX.out
                input[1] = file("${baseDir}/data/ref/ref.fasta")
                input[2] = file("${baseDir}/data/ref/ref.fasta.fai")
                input[3] = file("${baseDir}/data/ref/ref.dict")
                input[4] = file("${baseDir}/data/intervals.list")
                """
            }
        }

        then {
            assert process.success
            assert path(process.out[0][0][1]).readLines().contains('#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	NA12877')
            assert path(process.out[0][0][1]).readLines().contains('20	10040001	.	T	<NON_REF>	.	.	END=10040011	GT:DP:GQ:MIN_DP:PL	0/0:30:81:29:0,81,1025')
        }
    }

}
```

```bash
nf-test test modules/local/gatk/haplotypecaller/tests/main.nf.test
```

```bash
🚀 nf-test 0.8.4
https://code.askimed.com/nf-test
(c) 2021 - 2024 Lukas Forer and Sebastian Schoenherr


Test Process GATK_HAPLOTYPECALLER

  Test [86fd1bce] 'reads_son [bam]' PASSED (24.651s)
  Test [547788fd] 'reads_mother [bam]' PASSED (25.198s)
  Test [be786719] 'reads_father [bam]' PASSED (31.084s)


SUCCESS: Executed 3 tests in 80.942s
```

--------------------------------------------------------------------------------------------------------------------------------------------

## GATK_JOINTGENOTYPING Module Tests


* We are going to use an example here of using local inputs for a module test

```bash
modules/local/gatk/jointgenotyping/tests/inputs/
├── family_trio_map.tsv
├── reads_father.bam.g.vcf
├── reads_father.bam.g.vcf.idx
├── reads_mother.bam.g.vcf
├── reads_mother.bam.g.vcf.idx
├── reads_son.bam.g.vcf
└── reads_son.bam.g.vcf.idx
```

* main.nf.test

```groovy
nextflow_process {

    name "Test Process GATK_JOINTGENOTYPING"
    script "../main.nf"
    process "GATK_JOINTGENOTYPING"

    test("family_trio [vcf] [idx]") {

        when {
            params {
                outdir = "tests/results"
            }
            process {
                """
                input[0] = file("${baseDir}/modules/local/gatk/jointgenotyping/tests/inputs/family_trio_map.tsv")
                input[1] = "family_trio"
                input[2] = file("${baseDir}/data/ref/ref.fasta")
                input[3] = file("${baseDir}/data/ref/ref.fasta.fai")
                input[4] = file("${baseDir}/data/ref/ref.dict")
                input[5] = file("${baseDir}/data/intervals.list")
                """
            }
        }

        then {
            assert process.success
            assert path(process.out[0][0]).readLines().contains('#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	NA12877	NA12878	NA12882')
            assert path(process.out[0][0]).readLines().contains('20	10040772	.	C	CT	1568.89	.	AC=5;AF=0.833;AN=6;BaseQRankSum=0.399;DP=82;ExcessHet=0.0000;FS=4.291;MLEAC=5;MLEAF=0.833;MQ=60.00;MQRankSum=0.00;QD=21.79;ReadPosRankSum=-9.150e-01;SOR=0.510	GT:AD:DP:GQ:PL	0/1:14,16:30:99:370,0,348	1/1:0,17:17:51:487,51,0	1/1:0,25:25:75:726,75,0')
        }

    }

}

```

```bash
nf-test test modules/local/gatk/jointgenotyping/tests/main.nf.test
```


```bash
🚀 nf-test 0.8.4
https://code.askimed.com/nf-test
(c) 2021 - 2024 Lukas Forer and Sebastian Schoenherr


Test Process GATK_JOINTGENOTYPING

  Test [24c3cb4b] 'family_trio [vcf] [idx]' PASSED (18.915s)


SUCCESS: Executed 1 tests in 18.921s
```

