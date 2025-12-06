So in this one, let's dissect this very
interesting engineering blog from GitHub
called partitioning GitHub's relational
database to handle scale. This is how
the blog looks. You're one Google search
away from finding it. I'll link it in
the icard below and follow along if you
want to, but we'll dig very deep into
this blog to understand what they are
doing. It's some very interesting stuff
and how they have partitioned their
relational database. Now if they partner
with their relational database of course
they need to have a relational database
in the first place. So GitHub uses or
used to use a single massive MySQL
cluster to hold all the core data like
almost everything that you can imagine
was hold or was stored in this
particular MySQL cluster. Right now for
them to handle more load especially more
read request added replica standard
procedure and to handle or to manage the
connections well they put a proxy SQL in
front of it. Right? So proxy SQL is a
database proxy that accepts the
connection and then it routes the
request and does the connection pooling
and connection management on your
behalf. Right? Now when they had to
handle more scale they kept scaling that
kept scaling vertically. They did not go
for pre like uh prematurely optimizing
their infrastructure but after a certain
point this became a critical bottleneck.
So this bottleneck was critical because
they had a very high query volume of
almost a million requests per second
million not request but million queries
per second 950,000 queries per second
and
lots of problems. Problems being that
the tables were queried haphazardly
because there was no clear boundary on
who queries what. Anybody could query
anything. Anybody was joining on any
data. This lead to a noisy neighbor
problem that if load from one sort of
stuff. Let's say there are a lot of
hypothetical there are a lot of people
creating pull request at that time or a
lot of people starring it also affects
people accessing their commits. Right?
So that's a noisy neighbor problem
because there is everybody's on the same
underlying infrastructure literally the
same database load of one type of
request affects another type of request
and outages due to one affects everybody
right so these are scaling challenges
that come uh that GitHub was facing at
that time now the first question that
should come to your mind is why did they
not shard the database first of all not
every company that starts on day zero
shards the database nobody needs to
nobody over optimiz mises or nobody
overengineers it. But when the time
comes when you know the time comes when
you start seeing these kind of problems
but by the time you start seeing this as
a problem you already have a very
convoluted infrastructure or code base
where you are quering and joining
random tables. So there's no clear
boundary boundary that you can like
split it right and for them the existing
systems or the existing solutions did
not work. because they did not have a
clear ownership on who owns what, right?
Like cross-domain transactions were a
thing which means like you are you you
have a transaction spanning your stars
and your comments for example, right?
Not a good thing. So how did it solve
it? This is what the blog talks about.
So what GitHub did is GitHub did
two-tier strategy. In this video, by the
way, we'll dig deeper into how this
migrations really work. I'll go really
deep into it. It will be fascinating
stuff. So, GitHub uses two-tiered
strategy here. Now, what does two-tier
strategy mean? So, they did virtual
partitioning and physical partition. So,
virtual partitioning is all about
enforcing
the boundaries. They are not like they
just want to make sure that before they
do this physical separation they need to
make sure in that existing workflow
there is no crossdain communication that
happens right so they enforce boundaries
through called schema domains it's their
own tool that they built called schema
domains that all the tables belonging to
a schema this is not posgress schema
there's not that this is a schema domain
it's just a logical domain do which
encapsulates multiple tables and that
becomes a domain and all the query for
one domain should remain within that
domain. 

-----------------------------Before Jumma namaj ----------------
That's what they had to ensure
right and then comes physical
partitioning where they gradually
migrated the data. In this one, we'll
discuss how the migration happens,
right? Okay, let's dig deeper into
virtual partitioning via schema domains.
Simple concept, super simple concept
that they have a file called schema
domains. AML in which they define
logical schema like logical schemas like
gist and repositories and users in which
they contain the names of the tables
like just comments, gist and start gist
and repository contain issues, pull
request, repositories and so on and so
forth. So what they want is at the
application level there should not be a
single instance where a transaction is
quering just comments and pull requests
in the same in the same transaction. So
they're are in the same query or they're
joining this table. They want to make
sure that they want to highlight that if
the domain that they have enforced
has no violations which means this is
not talking this domain is not talking
to this domain then this domain is ready
to be pulled out of this cluster and put
into another cluster. Right. So
essentially this virtual partitioning
helped them without physical movement.
It helped them make made sure that a
domain is self-sufficient in answering
all sorts of queries and can be moved
out. That's what this is all about. How
did they do it? They used something they
built something called as query
lintters. This typically runs in the dev
and test environment. Prod environment.
We'll talk about it in 2 minutes. Right?
So in dev and test environment it's
simple query linders they look for the
queries they see if there are cross
domain joins in that case it raises the
it raises exceptions right and it raises
exceptions with very helpful messages of
course and this can be exempted by
adding a comment in the codebase. Now
this way just by measuring the number of
comments that are like this they need to
follow particular structure they know if
they have they they have complete
isolation for this domain or not right.
So query lters literally just looked at
the queries that are getting fired and
raised exceptions. This way they had to
take an exemption for it in the codebase
and bam. Right? But this way they knew
if a particular domain is ready to be
moved out because it is self-sufficient.
Now what about transaction like what
about production and transactions? So at
a query level you can do it. Look at a
select query and see if there are cross
if there are tables that belongs to
different domains and are joined. But
what about transactions? Transactions
may have different queries. So this
query may be may be bounded within one
domain. Another query may be bounded
within another domain. But these are
part of the same transaction. That's a
problem. So what they do is this is what
they do in production. They
make sure that again individual queries
are on schema but transactions can span
table. So the idea is very simple. What
they do is they capture all the queries
that are fired and then they analyze it.
when they analyze it, they make note of
it and they raise an alert. Hey, this is
what has happened. They don't block it,
but they raise an alert. So, if a
transaction actually includes queries to
tables that will move to separate
database, it will no longer able it will
no longer be able to guarantee
consistency. So, they raise alerts for
this, right? But then there are tables
which are shared. For example, let's say
reactions across your issues and PRs you
want to do it. So reaction is a table
that you can react on comments, you can
react on PRs, you can react everywhere.
So this reaction table is like a table
that spans multiple domains. What do you
do with this? Right? So for that they
essentially
they uh try to partition this table into
smaller tables each within each occupied
within that same domain. They did not
mention very specifics on what actually
did but it's more about they are
polymorphic tables splitting it and
moving it different places might work
and these such tables are exempted from
the process. So they don't raise errors
and exceptions for such tables. But in
reality when they had to do this
physical migration then they just made
sure that either they split the table
horizontally and place it if not then
they duplicate the data because it might
be just read only and write goes to one
press multiple they did not clear this
again out of scope for this discussion
but the idea is to make sure be prepared
for the physical migration.
That's the whole thing that I know that
for this domain all the queries that I'm
firing all the transactions they span
this one domain. So now we are ready to
move this domain into a different
cluster. Now let's dig deeper into how
the migration actually works. So the
second tier physical partitioning. So
what they do? So now here a domain is
virtually isolated. We are sure of it.
Right now in the in the entire
conversation beyond this, we will assume
that the gist domain is virtually
isolated. It was part of cluster A which
was the main primary cluster. It now
needs to move to cluster B. Right? So it
the gist domain gets its own cluster
because it's big enough. Now some things
two things that you certainly need to
know that these terms would appear.
First is proxy SQL. Proxy SQL is
essentially a database proxy. As I
mentioned in the beginning of the video,
it's a database proxy. Everybody
connects to proxy SQL. Proxy SQL
connects to MySQL servers, right? And
proxy SQL is specific to MySQL at the
moment. And Proxy SQL does connection
management, caching, routing, and
whatnot. So proxy SQL is a database
proxy, right? It sits in front of it. It
it sits in front of your database. Your
client talks to this, right? Okay. And
the second is GT. GTI is MySQL's global
transaction identifier. Treat this think
of this as a monotonically increasing
identifier for each transaction that is
committed. This is how wall works or or
rather this is how replication works
where your replica keeps track of which
GT the replica is in and which GT master
is in. So that it knows what's the
replica like between the two. Right?
Okay. Now setup. Now we'll go slow. So
setup is simple. So the setup looks like
this. I'll explain. So the setup is that
first what do you want to do? You want
to take just from cluster A and move it
to cluster B. So all tables of just
domain needs to be taken from cluster
and move to cluster B. So first thing
first they take the snapshot of cluster
A for these tables. Right? Then this
snapshot is loaded into cluster B.
Right? Nothing no traffic is going to
cluster B at this moment. Snapshot is
taken from cluster A. Snapshot is loaded
in cluster B. So cluster B is seeded
with some data. Right? Now
this snapshot is also loaded on replicas
of cluster B. So you have primary
cluster B and replicas of cluster B.
Right? So snapshot taken from cluster A
for those table and moved it to cluster
B and replicas of cluster B. Now the
replica of cluster B they replicate from
primary of cluster B. This is what is
happening right? So replication is set
up here. Then what happens is
replication is set up from primary of
cluster A to primary of cluster B. So
cluster B acts as replica for cluster A.
Here again request is not yet going. So
any changes that happens to cluster A
gets percolated to cluster B. Cluster B
applies it right. And any changes on
cluster B is taken to replicas. Right?
So any rights happening over here goes
to replica. So this way there is a lag
between cluster A, cluster B and replica
which is okay. And of course cluster A
has its own set of replicas. Right? Now
primary of B essentially becomes the
replica of A as we discussed. Now what
happens? There is proxy SQL setup for
both clusters cluster A and cluster B.
So proxy SQL is the point of entry for
any client who wants to talk to the
database. So anybody who wants to talk
to any node of cluster A talks to this
proxy SQL. Anybody who wants to talk to
cluster B talks to this proxy SQL.
Right? This is the setup. Now what they
do is reads and writes are redirected to
proxy SQL B. Reads and writes of what?
Gist. where in the main API server that
they would have they would say that hey
for gist use this connection object. So
any request for gist goes to proxy SQL
of cluster B and in proxy but B is still
not yet in sync with A right B is not
yet ready we just has the snapshot
loaded. So request that goes to proxy
SQL of B to handle gist. What are what
are we doing with that? So proxy SQL of
B is configured to route the request to
proxy SQL of A.
So request here that comes here for gist
because this is eventually going to
handle gist right. So request for gist
that came here goes here
to proxy SQL of cluster A. Now when
request goes to proxy SQL of cluster A.
Now what happens then it will go to
master or replica however because this
is capable of handling all things gist
right the data is moved but it's not yet
in sync right? So this way the first
step that is taken is request has
started coming to this proxy SQL and has
started going here to be handled. Once
your cluster B is fully ready all it has
to do is just switch traffic here.
That's it. Right. So step number one or
rather step number and whatever I was
numbered. So request for just started
coming to proxy SQL B. Proxy SQL B
redirects traffic to proxy SQL A. Like
literally this is the command that you
fire MySQL servers proxy SQL a MySQL 633
online it routes there right okay now
this has happened this is the setup that
I've mentioned that who follows who
cluster A has replica A1 replica A
replica A3 cluster A primary also rep
like cluster B primary replicates from
cluster A cluster B's replica replicates
from cluster B's primary literally this
diagram diagram is what I showed right.
Okay. Now, now this is happening. So
what is the current state? Read request
going here. Write request going here. No
problem. Right? Any rightes happening on
master of cluster A gets synced to
primary of cluster B. Here it gets
synced to replica of cluster B. So there
is an eventual consistency between these
systems. All good. Right? So now as in
when the traffic keeps hitting here you
will be in a position that this replica
is going to be catching up right now
there will come a time where the replica
lag is very small less than a second
right at that time what do you do you do
cut over
what is cut over
is your proxy equal b this which was
routing traffic here I want this to
route traffic here how would that happen
when would that happen? So that is what
the cut over is. So I'm cutting over
traffic here and moving it here. So
let's go step by step what it does. So
you enable the read only mode on primary
of A. Right? So primary of A, read only
mode array. What would happen to
existing request? They would start
failing. Right? So read only mode
enabled on primary of A. You prevent all
requests that are going to A and B. They
both are unable to handle it. Right?
Perfectly fine. All res all request will
all request during this time during the
cut over time will result in a 5xx
completely. Okay. Now what you have to
do is you have to measure the GT
difference between cluster A and cluster
B like how far behind is cluster B from
cluster A and then let it catch up. You
keep pulling cluster B to see does it
have all the changes of cluster A? Does
it have all the changes of cluster A. So
you just measure the difference between
GT ID between the two. So you know if
because you have stopped all the rights
no no no new commits are happening on
this cluster A. So eventually B will
catch up. The moment B catches up you
stop replication that you stop B
replicating from A because now it has
all got up. So you stop the replication
of B. As soon as you stop B replicating
from A, you update the proxy SQL
configuration to now send traffic to B's
primary here. So you are standing here.
Now it is all caught up. Now you start
sending start sending traffic here. And
then done. Now your B cluster is all
sync with cluster A. No rightes have
happened. Some requests have failed but
how many? We'll check. Some requests
have failed but now B is self-sufficient
to handle all things just and request
started going to B.
How long does the six steps take? The
six steps take less than 100
milliseconds to complete. It sounds too
much but it doesn't take a long time.
Machines are lightning fast at this
moment. It doesn't take long time but
again the moment you start your cut over
is when you know your application lag is
less than a second otherwise you don't
start. So you wait for your cluster B to
keep catching up with cluster A and when
the time is right which means less than
1 second or less than half a second
that's when you trigger your cut over
that would be your downtime availability
time out but not much right but this is
how your GitHub did this migration now
this I just explained for just imagine
for other domains like repositories and
users and this and that they keep giving
them individual clusters they repeated
the same process several times for each
of the logical schema the virtual
partition that they had. Pretty
fascinating. Pretty fascinating, right?
Again, this was what we also used to do.
This was a bread and butter. Yeah, being
an S sur you know a lot of this stuff.
So, I had my good two two and a half
year stint at two different places in S.
But this is very fascinating. I would
highly recommend you to mimic all of
this locally. It does not take a lot of
time with LLMs. It certainly would not.
Right? Please use LLM wisely. Build a
prototype and build an understanding of
how this no downtime database migrations
and cut overs and all happen. You have
all the theory here. You have some
commands that would give you an idea on
what needs to happen. But I would highly
highly highly highly recommend you to
implement this. And yeah, this is all
what I wanted to cover in this one. I
hope you found it interesting. Hope you
found it amusing. That's it for this
one. I'll see you in the next one.
Thanks a
[Music]