# This requires all the details which is required to build customized frds image

You need to make the base image and setup in a way so that kubernetes can access it locally or the private repo 

      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: raining-lab-depl-1
      spec:
        replicas: 2
        selector:
          matchLabels:
            raining-lab-pod: raining-lab-pod-label-1
        template:
          metadata:
            name: raining-lab-pod
            labels:
              raining-lab-pod: raining-lab-pod-label-1
          spec:
            containers:
              - name: raining-lab-pod-container-1
                image: frds:1.0
             ***imagePullPolicy: Never***
                ports:
                  - containerPort: 1389
                env:
                  - name: USE_DEMO_KEYSTORE_AND_PASSWORDS
                    value: "true"
                command: ["sh","-c"]
                args: ["start-ds"]

 
 
          
          
         
