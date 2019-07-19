pipeline {
    agent {
        node "master"
    }
    environment {
        env = "dev"
    }
    parameters {
        choice(name:"FORMAT", choices:['tar','zip','rar'], description:"Formatos")
    }
    stages {
        stage ("Compilação") {
            agent {
                docker {
                    image "centos:7"
                }
            }
            steps {
				echo "Continuar pipeline ?"
			}
        }
        stage ("Validar"){
			input {
				message "Continuar"
			}
			steps{
				echo "Continuar pipeline ?"
			}
		}
        stage ("Batman") {
            steps {
                echo "EITA"
            }
        }
        stage ("oh yeah") {
            steps {
                echo "CORINGA"
            }
        }
        stage("Deploy"){
			parallel{
				stage("Linux"){
					steps{
						echo "Deploy sera em: $ENV"
					}
				}

				stage("Windows"){
					steps{
						echo "Executando no Windows XP"
					}
				}
			}
		}
		stage ("Parametros"){
		    steps {
		        echo "${params.FORMAT}"
		    }
		}
    }
}
