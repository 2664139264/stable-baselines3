.. _her:

.. automodule:: stable_baselines3.her


HER
====

`Hindsight Experience Replay (HER) <https://arxiv.org/abs/1707.01495>`_

HER is an algorithm that works with off-policy methods (DQN, SAC, TD3 and DDPG for example).
HER uses the fact that even if a desired goal was not achieved, other goal may have been achieved during a rollout.
It creates "virtual" transitions by relabeling transitions (changing the desired goal) from past episodes.


.. warning::

  Starting from Stable Baselines3 v1.1.0, ``HER`` is no longer a separate algorithm
  but a replay buffer class ``HerReplayBuffer`` that must be passed to an off-policy algorithm
  when using ``MultiInputPolicy`` (to have Dict observation support).

.. warning::

    HER requires the environment to follow the legacy `gym_robotics.GoalEnv interface <https://github.com/Farama-Foundation/Gymnasium-Robotics/blob/a35b1c1fa669428bf640a2c7101e66eb1627ac3a/gym_robotics/core.py#L8>`_
    In short, the ``gym.Env`` must have:
    - a vectorized implementation of ``compute_reward()``
    - a dictionary observation space with three keys: ``observation``, ``achieved_goal`` and ``desired_goal``


.. warning::

  Because it needs access to ``env.compute_reward()``
  ``HER`` must be loaded with the env. If you just want to use the trained policy
  without instantiating the environment, we recommend saving the policy only.


.. note::

  Compared to other implementations, the ``future`` goal sampling strategy is inclusive:
  the current transition can be used when re-sampling.


Notes
-----

- Original paper: https://arxiv.org/abs/1707.01495
- OpenAI paper: `Plappert et al. (2018)`_
- OpenAI blog post: https://openai.com/blog/ingredients-for-robotics-research/


.. _Plappert et al. (2018): https://arxiv.org/abs/1802.09464

Can I use?
----------

Please refer to the used model (DQN, QR-DQN, SAC, TQC, TD3, or DDPG) for that section.

Example
-------

This example is only to demonstrate the use of the library and its functions, and the trained agents may not solve the environments. Optimized hyperparameters can be found in RL Zoo `repository <https://github.com/DLR-RM/rl-baselines3-zoo>`_.

.. code-block:: python

    from stable_baselines3 import HerReplayBuffer, DDPG, DQN, SAC, TD3
    from stable_baselines3.her.goal_selection_strategy import GoalSelectionStrategy
    from stable_baselines3.common.envs import BitFlippingEnv

    model_class = DQN  # works also with SAC, DDPG and TD3
    N_BITS = 15

    env = BitFlippingEnv(n_bits=N_BITS, continuous=model_class in [DDPG, SAC, TD3], max_steps=N_BITS)

    # Available strategies (cf paper): future, final, episode
    goal_selection_strategy = "future" # equivalent to GoalSelectionStrategy.FUTURE

    # Initialize the model
    model = model_class(
        "MultiInputPolicy",
        env,
        replay_buffer_class=HerReplayBuffer,
        # Parameters for HER
        replay_buffer_kwargs=dict(
            n_sampled_goal=4,
            goal_selection_strategy=goal_selection_strategy,
        ),
        verbose=1,
    )

    # Train the model
    model.learn(1000)

    model.save("./her_bit_env")
    # Because it needs access to `env.compute_reward()`
    # HER must be loaded with the env
    model = model_class.load("./her_bit_env", env=env)

    obs, info = env.reset()
    for _ in range(100):
        action, _ = model.predict(obs, deterministic=True)
        obs, reward, terminated, truncated, _ = env.step(action)
        if terminated or truncated:
            obs, info = env.reset()


Results
-------

This implementation was tested on the `parking env <https://github.com/eleurent/highway-env>`_
using 3 seeds.

The complete learning curves are available in the `associated PR #120 <https://github.com/DLR-RM/stable-baselines3/pull/120>`_.



How to replicate the results?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Clone the `rl-zoo repo <https://github.com/DLR-RM/rl-baselines3-zoo>`_:

.. code-block:: bash

  git clone https://github.com/DLR-RM/rl-baselines3-zoo
  cd rl-baselines3-zoo/


Run the benchmark:

.. code-block:: bash

  python train.py --algo tqc --env parking-v0 --eval-episodes 10 --eval-freq 10000


Plot the results:

.. code-block:: bash

  python scripts/all_plots.py -a tqc -e parking-v0 -f logs/ --no-million


Parameters
----------

HER Replay Buffer
-----------------

.. autoclass:: HerReplayBuffer
  :members:
  :inherited-members:


Goal Selection Strategies
-------------------------

.. autoclass:: GoalSelectionStrategy
  :members:
  :inherited-members:
    :undoc-members:
