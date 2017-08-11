In this document I will explain what type of steering behaviours I have used in my football game

Football Game

This game automatically lets two teams fight for the earliest goal, the game restarts once a goal has been made.

Steering behaviors:

How logic gets called for the steering behaviors
The character script contacts the Steering Core component and call ‘Change Direction’. 
This will notify the steering system to start moving towards the given transform. In this case, it’s the goal or the ball. This depends on the current behavior.
  IEnumerator ObtainedBallBehaviour()
    {
        while (behaviourState == Enums.CharacterBehaviour.SeekingGoal)
        {
            steeringCore.ChangedDirection(enemyGoal.transform);
            steeringCore.m_MaxSpeed = 5f;

            float thisDistanceToGoal = GetDistanceToGoal();

            /* Shoot at the goal if the character is near enough */
            if (thisDistanceToGoal < ballGoalShootDistance)
            {
                ball.Shoot(enemyGoal.transform.position, ballShootForce);
            } else
            {
                TryShootToCloserTeamMember(thisDistanceToGoal);
            }

            yield return new WaitForSeconds(0.25f);
        }
    }

    IEnumerator SeekBallBehaviour ()
    {
        while (behaviourState == Enums.CharacterBehaviour.SeekingBall)
        {
            steeringCore.ChangedDirection(ball.transform);
            steeringCore.m_MaxSpeed = 6f;

            yield return new WaitForSeconds(0.25f);
        }
    }



Seeker
Tries to get the ball at all costs, it directly uses the Changed Direction and starts seeking it with desired velocity

    if (SteeringCore == null)
            {
                return;
            }

            if (SteeringCore.Target == null)
            {
                return;
            }

            // Calculate desired velocity
            m_DesiredVelocity = (SteeringCore.Target - transform.position).normalized * SteeringCore.MaxSpeed;

            // Calculate steering force
            SteeringForce = m_DesiredVelocity - SteeringCore.Velocity;
        }

Wanderer
Walks around his own area based on the given ‘wander radius’ and distance. 
This behavior also has a chance of making a goal, since it can sometimes get near a goal.
And trigger the Shoot function, as a wanderer it can also intercept the ball if it goes to wards the opposite team player with the ball.


            // Calculate desired velocity
            m_WanderCirclePosition = transform.position + transform.forward * m_WanderDistance;
            /*
            m_WanderOffset = transform.forward * m_WanderRadius;
            m_WanderOffset = Quaternion.AngleAxis(m_WanderAngle, Vector3.up) * m_WanderOffset;
            m_WanderAngle += Random.Range(-m_WanderAngleChange, m_WanderAngleChange);
            */
            m_DesiredVelocity = (m_RandomPoint - transform.position).normalized * SteeringCore.MaxSpeed;
            //m_DesiredVelocity = (m_WanderCirclePosition + m_WanderOffset - transform.position).normalized * SteeringCore.MaxSpeed;

            // Calculate steering force
            SteeringForce = m_DesiredVelocity - SteeringCore.Velocity;

            // Update timer
            UpdateTimer();

Pathfinding
The pathfinding uses a steering path component, this is attached in the inspector.
Walks to specific set of points.
 public override void PerformSteeringBehavior()
        {
            if (SteeringCore == null || m_PathToFollow == null)
            {
                return;
            }

            if (m_PathToFollow.PointList.Count <= 1)
            {
                return;
            }

            // Get nearest point on path
            Vector3 nearestPoint;
            Vector3 previousPoint;
            Vector3 nextPoint;
            Vector3 pathLine;
            Vector3 normalPointA;
            Vector3 normalPointB;

            GetNearestPointOnPath(out nearestPoint, out previousPoint, out nextPoint);
            normalPointA = GetNormalPoint(previousPoint, nearestPoint);
            normalPointB = GetNormalPoint(nearestPoint, nextPoint);

            if (previousPoint == Vector3.zero)
            {
                normalPointA = normalPointB;
            }

            Vector3 predictedPosition = transform.position + SteeringCore.Velocity;
            m_NormalPoint = GetNearestPoint(normalPointA, normalPointB);
            Vector3 normal = m_NormalPoint - predictedPosition;

            if (m_NormalPoint == normalPointA)
            {
                pathLine = nearestPoint - previousPoint;
            }
            else
            {
                pathLine = nextPoint - nearestPoint;
            }

            if (normal.sqrMagnitude > m_PathToFollow.Radius * m_PathToFollow.Radius)
            {
                // Calculate desired velocity
                m_DesiredVelocity = (m_NormalPoint - transform.position).normalized * SteeringCore.MaxSpeed;

                // Calculate steering force
                SteeringForce = m_DesiredVelocity - SteeringCore.Velocity;
            }
            else if ((normal * m_NormalScaleOnAnticipation).sqrMagnitude > m_PathToFollow.Radius * m_PathToFollow.Radius)
            {
                // Calculate desired velocity
                m_DesiredVelocity = (m_NormalPoint - transform.position).normalized * SteeringCore.MaxSpeed;

                // Calculate steering force
                SteeringForce = m_DesiredVelocity - SteeringCore.Velocity;
                SteeringForce *= m_SteeringForceScaleOnAnticipation;
            }
            else
            {
                SteeringForce = Vector3.zero;
            }

Configurable components:

Goal 
- Side (The characters will automatically search for the goal on the opposite side)
- LevelToLoadOnGoal (This level will be loaded once it has been hit)
  /* If object of tag ball hits the goal, restart the game */
    private void OnTriggerEnter(Collider collider)
    {
        if (collider.transform.tag == "Ball")
            SceneManager.LoadScene(levelToLoadOnGoal);
    }
}


Character
- Team Side (This is to let the character know which goal to seek)
- Goal Shoot Distance (Amount of distance required to start shooting a goal)

    public float GetDistanceToGoal ()
    {
        return Vector3.Distance(this.transform.position, enemyGoal.transform.position);
    }

    private void TryShootToCloserTeamMember(float thisDistanceToGoal)
    {
        for (int i = 0; i < teamPlayers.Count; i++)
        {
            if (teamPlayers[i].GetDistanceToGoal() < thisDistanceToGoal)
            {
                this.transform.LookAt(teamPlayers[i].transform);
                ball.Shoot(teamPlayers[i].transform.position, ballShootForce);

Behavior Seek
- Blend Scale (How much this component blends with other Steering Behavior components)

Behavior Wander:
- Blend Scale (How much this component blends with other Steering Behavior components)
- Wander Radius (How big the selection area is for the next area)
- Wander Distance (Distance between the character and the next point)
- Random Point Frequency (Time it takes to make a new wander point)

Behavior Path:
- Blend Scale (How much this component blends with other Steering Behavior components)
- Path to follow (Insert a Steering Path, and it will iterate over all the points)
- Normal scale (Anticipation distance to create smooth point transitions)
- Steering force scaling (How much force is applied to steering)


Character component:
All required components are obtained in Awake()
Also the goal and all team members are obtained here.

At Awake the character subscribes to On Owner Change.
This is to get a callback for when the ball owner has been changed,
this is used to set the state of the character.

(Characters subscribe to the current ball in the scene, 
if it has a owner change it will call a method that changes the behavior of the character)


(Once the ball owner has changed, I check if the character owner equals current object, if so then go seek the goal, else seek the ball)


Ball Component
(Code of event being called once the owner of the ball changes, this sends On Owner Change towards all characters. If the ball has no owner, it will also enable physics. The reason for this is for the Shoot function.)
(Used when the player is near a goal, it will try to force the ball into the goal)

 public void SetOwner(Character _owner)
    {
        if (_owner != null)
        {
            transform.position = _owner.BallSlot.position;
            transform.SetParent(_owner.BallSlot);
        }
        else
        {
            transform.SetParent(null);
        }

        owner = _owner;
        OnOwnerChange(owner);
    }

    public void Shoot(Vector3 target, float force)
    {
        SetOwner(null);
        rigidBody.AddForce((target - this.transform.position) * force, ForceMode.Impulse);
    }

}




Football game references: 

https://designagame.eu/2015/04/making-a-sport-game-in-unity-model-prototype/

https://www.red3d.com/cwr/steer/gdc99/

programming in AI by example by Matt Buckland# GAMING-AI-CW
GAMING AI  COURSEWORK 
