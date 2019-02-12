SVN working notes
==================

Merging dcc_group -> users
--------------------------

Before merging make sure that all your work is commited to the server::

    svn commit -m "status before merge"   (or using CLion)

Then, changes in dcc_group branch can be merged to user branches in usual way::

    svn merge ^/branches/dcc_group    (or using CLion)

Merging users -> dcc_group
--------------------------

Changes from user branches to dcc_group, should be done using the following procedure in order to keep the branch alive and avoid future "duplicate commit" conflicts:

1. Merge to dcc_group using reintegrate option::

    svn merge --reintegrate ^/branches/user           [At dcc_group branch] 

2. Commit the merged version::

    svn commit -m "merged from user"                 [At dcc_group branch] 

3. Check the revision number of the commit associated to the merge::

    Committed revision XXX. 

4. Merge this specific revision only using record-only  (http://svnbook.red-bean.com/en/1.7/svn.branchmerge.advanced.html) at the user branch using command line::

    svn update 
    svn merge --record-only -c XXX ^/branches/dcc_group   [At user branch]

5. Commit  merged revision at the user branch::

    svn commit -m "merged record"                         [At user branch] 

After this procedure dcc_group can be updated and modified and the changes can be merged into user branches as usual.

Merge trunk -> dcc_group
------------------------
To merg from trunk to group make sure that you are in dcc_group branch, then execute::

    svn merge --accept=postpone ^/trunk 

Sometimes after a big merge some connection issues may apear::

    svn: E120106: ra_serf: The server sent a trunkated HTTP response body

To fix it just cleanup and try again merge::

    svn cleanup
    svn up

Make sure to resolve all conflix that may apear::

    svn resolve


You can check that everything is Ok by::

    svn status


Merging dcc_group -> trunk
--------------------------

Merging from dcc_group to trunk branches should be done carefully using the queue system

1. Make sure that qcc_group branch contains all the changes from trunk (see Merge trunk -> dcc_group section)

2. Use Qchem queue systems (check qchem dev wiki) to merge dcc_group into trunk

After merge from dcc_group branch to trunk using the queue system, dcc_group branch will be deleted automatically. dcc_group should be recreated from trunk::

    svn copy ^/trunk  ^/branches/dcc_group -m "recreate dcc_group branch"

Already checkout local directories (from dcc_group branch) should still work normally and can be updated as usual using::

    svn update

User branches should merge changes from recreated dcc_group!!! If troubles appear with duplicated commited files appear, they can be resolved by using::

    svn resolve —accept=working  file_in_conflict


SVN scheme
----------

.. image :: images/svn_scheme.pdf
    :align: center

