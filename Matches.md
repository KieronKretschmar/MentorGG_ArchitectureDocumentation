## Publishing process
- Updating MatchEntities
    - MatchEntities is referenced by multiple projects, as it holds the definition for the MatchDataSets being transferred to MatchDBI and more, and also is the Code First basis for the MatchDB schema.
    - Consumers of MatchDataSets (MatchDBI, SituationOperator) should only accept MatchDataSets of one major version.
    

    - Publishing major updates (breaking changes) to MatchEntities should follow this pattern:
        1. Update DemoFileWorker so that it publishes MatchDataSets of newer version.
        2. Wait until all MatchDataSets of the previous version are consumed.
        3. Update consumers (including MatchDBI and therefore the MatchDB using migrations) so that they only accept MatchDataSets of the newer version.
        
        If, for some reason, MatchDataSets of the older version appear at consumers after Step 3, the entire analysis of the match should be restarted beginning with (the new) DemoFileWorker.

