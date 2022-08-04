FROM node:lts-alpine

WORKDIR /app

ENV NODE_ENV production

RUN adduser --system --uid 1001 nextjs

COPY public ./public
COPY .next/standalone .
COPY .next/static ./.next/static

RUN chown -R 1001:0 /app \
  && chmod -R g+=wrx /app

RUN ls -ltra /app

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["npm","run", "start"]
