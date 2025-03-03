# syntax=docker/dockerfile:1.3
#--------------------------------------------
# Dev Stage - Assembles and Builds Frontend
#--------------------------------------------
    FROM --platform=$BUILDPLATFORM node:18-alpine as dev

    ARG APP_SHELL_VERSION=next
    
    RUN mkdir -p /app
    WORKDIR /app
    
    COPY packages/openmrs-esm-primary-navigation-app-6.1.0-trigonal.tgz .
    COPY packages/openmrs-esm-login-app-6.1.0-trigonal.tgz .
    COPY packages/openmrs-esm-patient-chart-app-9.2.1-trigonal.tgz .
    
    COPY spa-assemble-config.json .
    COPY spa-build-config.json .
    
    # RUN npm install -g npm@latest
    
    RUN yarn add ./openmrs-esm-primary-navigation-app-6.1.0-trigonal.tgz
    RUN yarn add ./openmrs-esm-login-app-6.1.0-trigonal.tgz
    RUN yarn add ./openmrs-esm-patient-chart-app-9.2.1-trigonal.tgz
    
    # RUN npm install -g npm@10.9.7
    
    ARG CACHE_BUST
    RUN npx --legacy-peer-deps openmrs@${APP_SHELL_VERSION:-next} assemble --manifest --mode config --config spa-assemble-config.json --target ./spa
    RUN npx --legacy-peer-deps openmrs@${APP_SHELL_VERSION:-next} build --build-config spa-build-config.json --target ./spa
    RUN if [ ! -f ./spa/index.html ]; then echo 'Build failed. Please check the logs above for details. This may have happened because of an update to a library that OpenMRS depends on.'; exit 1; fi
    
    #--------------------------------------------
    # Runtime Stage - Published Image
    #--------------------------------------------
    FROM nginx:1.25-alpine
    
    RUN apk update && \
        apk upgrade && \
        # add more utils for sponge and envsubst
        apk add --no-cache moreutils
    
    # clear any default files installed by nginx
    RUN rm -rf /usr/share/nginx/html/*
    
    COPY startup.sh /usr/local/bin/startup.sh
    RUN chmod +x /usr/local/bin/startup.sh
    
    COPY nginx.conf /etc/nginx/nginx.conf
    
    COPY --from=dev /app/spa /usr/share/nginx/html
    COPY config-core_demo.json /usr/share/nginx/html
    
    #Replace logo
    COPY logo.png /usr/share/nginx/html
    COPY favicon.ico /usr/share/nginx/html
    # COPY index.html /usr/share/nginx/html
    
    CMD ["/usr/local/bin/startup.sh"]
    