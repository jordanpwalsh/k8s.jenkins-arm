node {
    checkout scm

    sh "git rev-parse --short HEAD > commit-id"
    tag = readFile('commit-id').replace("\n", "").replace("\r", "")

    sh "arch > arch"
    arch = readFile('arch').trim()

    appName = "jenkins"
    registryHost = "127.0.0.1:30035" //this is actual internal registry we're exposing with a nodeport
    
    if(arch=="x86_64"){
        imageName = "${registryHost}/${appName}"
    }
    else {
        imageName = "${registryHost}/${appName}-${arch}"
    }
    env.BUILDIMG=imageName

    stage("Container Build") {
    
        sh "docker build -t ${imageName}:latest -f Dockerfile ."
        sh "docker tag ${imageName}:latest ${imageName}:${tag}"
    }
    stage("Container Push") {

        //withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'nexusPassword', usernameVariable: "nexusUser")]) {
            //sh "docker login ${registryHost} -u ${env.nexusUser} -p ${nexusPassword}"
            sh "docker push ${imageName}:latest"
            sh "docker push ${imageName}:${tag}"
            

        //}
    }

    stage("Container Deploy") {
        withKubeConfig([credentialsId: 'kubeconfig', serverUrl: "https://kubernetes.default", contextName: 'kubernetes-admin@kubernetes', clusterName: 'kubernetes']) { //default is namespace
	        sh "kubectl apply -f k8s-app.yaml"
            sh "kubectl set image deployment/${appName} ${appName}=${imageName}:${tag} --record"
        }
    }

    //we dont want to actually deploy jenkins becuase this is running in jenkins, just keep image updated
    //  stage("Pod Rollout") {
    //     withKubeConfig([credentialsId: 'kubeconfig', serverUrl: "https://kubernetes.default", contextName: 'kubernetes-admin@kubernetes', clusterName: 'kubernetes']) { //default is namespace
    //         sh "kubectl rollout status deployment/${appName}"
    //     }
    // }
}
