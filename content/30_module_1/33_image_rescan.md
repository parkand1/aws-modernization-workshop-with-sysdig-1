---
title: "Modify the image and trigger a second scan"
chapter: false
weight: 33
---

For illustration purposes, let's rebuild our image and make it more secure by starting with a different Base image. To do so:

1. Go back into Cloud9 Workspace
2. Edit the `Dockerfile` and in the first line update the base image from

	```
	FROM node:12
	```

	to
	```
	FROM bitnami/node:12
	```

	The file should look like this

	```
	FROM bitnami/node:12

	# Create app directory
	WORKDIR /usr/src/app

	# Install app dependencies
	# A wildcard is used to ensure both package.json AND package-lock.json are copied
	# where available (npm@5+)
	COPY package*.json ./

	RUN npm install
	# If you are building your code for production
	# RUN npm ci --only=production

	# Bundle app source
	COPY . .

	EXPOSE 8080
	CMD [ "node", "server.js" ]
	```

3. Now rebuild and push the image again with:

	```
	docker build . -t $IMAGE
	docker push $IMAGE
	```

	The image will automatically be scanned, as before.

	**Troubleshooting** If the build fails reporting no space left on device, then you can delete the older `node` image, as follows.

	```
	docker rmi $(docker images | grep node | grep -v lambda | awk '{print $3}')
	```



4. Once completed, you will see that the scan result now shows a more recent image (based on debian/10) with fewer vulnerabilities. ![Sysdig Secure](/images/30_module_1/securescann03.png)
