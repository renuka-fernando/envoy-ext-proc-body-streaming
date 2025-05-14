FROM alpine:3.21.0

USER 10014

COPY ./external_processor /bin/external_processor
EXPOSE 9001
ENTRYPOINT ["/bin/external_processor"]
