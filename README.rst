A task scheduler built on top of redis. It stores all jobs and schedules in redis which provides
persistency, distributed work and fault tolerance.


Installation
------------

You can install rescheduler with pip:

.. code-block:: console

    $ pip install rescheduler


Quickstart
----------

Server side:
~~~~~~~~~~~~

.. code-block:: python

    import asyncio
    import aioredis
    from rescheduler import Scheduler, Job


    def tick():
        print("tick")


    def tack():
        print("tack")


    async def callback(job: Job):
        if job.data['method'] == 'tick':
            tick()

        elif job.data['method'] == 'tack':
            tack()


    async def main():
        conn_pool = await aioredis.create_redis_pool(('localhost', 6379))
        async with Scheduler(conn_pool=conn_pool, job_callback=callback, use_keyspace_notifications=True):
            await asyncio.sleep(30)

        conn_pool.close()
        await conn_pool.wait_closed()


    if __name__ == '__main__':
        asyncio.run(main())


Client side:
~~~~~~~~~~~~

.. code-block:: python

    import asyncio
    import aioredis
    from rescheduler import Scheduler, Job, CronTrigger


    async def main():
        conn_pool = await aioredis.create_redis_pool(('localhost', 6379))
        scheduler = Scheduler(conn_pool=conn_pool, job_callback=lambda: None)

        await scheduler.add_job(
            Job(
                id='tick-task',
                trigger=CronTrigger.parse(expr='*/10 * * * * *', seconds_ext=True),
                data={'method': 'tick'},
            )
        )

        await scheduler.add_job(
            Job(
                id='tack-task',
                trigger=CronTrigger.parse(expr='*/10 * * * * *', seconds_ext=True),
                data={'method': 'tack'},
            ),
            delay=5.0,
        )

        conn_pool.close()
        await conn_pool.wait_closed()


    if __name__ == '__main__':
        asyncio.run(main())
