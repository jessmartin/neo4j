ClassMethods
============






.. toctree::
   :maxdepth: 3
   :titlesonly:


   

   

   

   

   

   

   

   

   

   

   

   

   

   

   

   

   

   

   

   

   




Constants
---------





Files
-----



  * `lib/neo4j/active_node/labels.rb:79 <https://github.com/neo4jrb/neo4j/blob/master/lib/neo4j/active_node/labels.rb#L79>`_





Methods
-------



.. _`Neo4j/ActiveNode/Labels/ClassMethods#base_class`:

**#base_class**
  

  .. code-block:: ruby

     def base_class
       unless self < Neo4j::ActiveNode
         fail "#{name} doesn't belong in a hierarchy descending from ActiveNode"
       end
     
       if superclass == Object
         self
       else
         superclass.base_class
       end
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#blank?`:

**#blank?**
  

  .. code-block:: ruby

     def empty?
       !self.all.exists?
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#constraint`:

**#constraint**
  Creates a neo4j constraint on this class for given property

  .. code-block:: ruby

     def constraint(property, constraints)
       Neo4j::Session.on_session_available do |session|
         unless Neo4j::Label.constraint?(mapped_label_name, property)
           label = Neo4j::Label.create(mapped_label_name)
           drop_index(property, label) if index?(property)
           label.create_constraint(property, constraints, session)
         end
       end
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#count`:

**#count**
  

  .. code-block:: ruby

     def count(distinct = nil)
       fail(InvalidParameterError, ':count accepts `distinct` or nil as a parameter') unless distinct.nil? || distinct == :distinct
       q = distinct.nil? ? 'n' : 'DISTINCT n'
       self.query_as(:n).return("count(#{q}) AS count").first.count
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#delete_all`:

**#delete_all**
  Deletes all nodes and connected relationships from Cypher.

  .. code-block:: ruby

     def delete_all
       self.neo4j_session._query("MATCH (n:`#{mapped_label_name}`) OPTIONAL MATCH n-[r]-() DELETE n,r")
       self.neo4j_session._query("MATCH (n:`#{mapped_label_name}`) DELETE n")
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#destroy_all`:

**#destroy_all**
  Returns each node to Ruby and calls `destroy`. Be careful, as this can be a very slow operation if you have many nodes. It will generate at least
  one database query per node in the database, more if callbacks require them.

  .. code-block:: ruby

     def destroy_all
       all.each(&:destroy)
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#drop_constraint`:

**#drop_constraint**
  

  .. code-block:: ruby

     def drop_constraint(property, constraint = {type: :unique})
       Neo4j::Session.on_session_available do |session|
         label = Neo4j::Label.create(mapped_label_name)
         label.drop_constraint(property, constraint, session)
       end
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#drop_index`:

**#drop_index**
  

  .. code-block:: ruby

     def drop_index(property, label = nil)
       label_obj = label || Neo4j::Label.create(mapped_label_name)
       label_obj.drop_index(property)
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#empty?`:

**#empty?**
  

  .. code-block:: ruby

     def empty?
       !self.all.exists?
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#exists?`:

**#exists?**
  

  .. code-block:: ruby

     def exists?(node_condition = nil)
       unless node_condition.is_a?(Integer) || node_condition.is_a?(Hash) || node_condition.nil?
         fail(InvalidParameterError, ':exists? only accepts ids or conditions')
       end
       query_start = exists_query_start(node_condition)
       start_q = query_start.respond_to?(:query_as) ? query_start.query_as(:n) : query_start
       start_q.return('COUNT(n) AS count').first.count > 0
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#find`:

**#find**
  Returns the object with the specified neo4j id.

  .. code-block:: ruby

     def find(id)
       map_id = proc { |object| object.respond_to?(:id) ? object.send(:id) : object }
     
       result =  if id.is_a?(Array)
                   find_by_ids(id.map { |o| map_id.call(o) })
                 else
                   find_by_id(map_id.call(id))
                 end
       fail Neo4j::RecordNotFound if result.blank?
       result
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#find_by`:

**#find_by**
  Finds the first record matching the specified conditions. There is no implied ordering so if order matters, you should specify it yourself.

  .. code-block:: ruby

     def find_by(values)
       all.where(values).limit(1).query_as(:n).pluck(:n).first
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#find_by!`:

**#find_by!**
  Like find_by, except that if no record is found, raises a RecordNotFound error.

  .. code-block:: ruby

     def find_by!(values)
       find_by(values) || fail(RecordNotFound, "#{self.query_as(:n).where(n: values).limit(1).to_cypher} returned no results")
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#find_each`:

**#find_each**
  

  .. code-block:: ruby

     def find_each(options = {})
       self.query_as(:n).return(:n).find_each(:n, primary_key, options) do |batch|
         yield batch.n
       end
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#find_in_batches`:

**#find_in_batches**
  

  .. code-block:: ruby

     def find_in_batches(options = {})
       self.query_as(:n).return(:n).find_in_batches(:n, primary_key, options) do |batch|
         yield batch.map(&:n)
       end
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#first`:

**#first**
  Returns the first node of this class, sorted by ID. Note that this may not be the first node created since Neo4j recycles IDs.

  .. code-block:: ruby

     def first
       self.query_as(:n).limit(1).order(n: primary_key).pluck(:n).first
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#index`:

**#index**
  Creates a Neo4j index on given property
  
  This can also be done on the property directly, see Neo4j::ActiveNode::Property::ClassMethods#property.

  .. code-block:: ruby

     def index(property, conf = {})
       Neo4j::Session.on_session_available do |_|
         drop_constraint(property, type: :unique) if Neo4j::Label.constraint?(mapped_label_name, property)
         _index(property, conf)
       end
       indexed_properties.push property unless indexed_properties.include? property
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#index?`:

**#index?**
  

  .. code-block:: ruby

     def index?(index_def)
       mapped_label.indexes[:property_keys].include?([index_def])
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#indexed_properties`:

**#indexed_properties**
  

  .. code-block:: ruby

     def indexed_properties
       @_indexed_properties ||= []
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#last`:

**#last**
  Returns the last node of this class, sorted by ID. Note that this may not be the first node created since Neo4j recycles IDs.

  .. code-block:: ruby

     def last
       self.query_as(:n).limit(1).order(n: {primary_key => :desc}).pluck(:n).first
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#length`:

**#length**
  

  .. code-block:: ruby

     def count(distinct = nil)
       fail(InvalidParameterError, ':count accepts `distinct` or nil as a parameter') unless distinct.nil? || distinct == :distinct
       q = distinct.nil? ? 'n' : 'DISTINCT n'
       self.query_as(:n).return("count(#{q}) AS count").first.count
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#mapped_label`:

**#mapped_label**
  

  .. code-block:: ruby

     def mapped_label
       Neo4j::Label.create(mapped_label_name)
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#mapped_label_name`:

**#mapped_label_name**
  

  .. code-block:: ruby

     def mapped_label_name
       @mapped_label_name || label_for_model
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#mapped_label_names`:

**#mapped_label_names**
  

  .. code-block:: ruby

     def mapped_label_names
       self.ancestors.find_all { |a| a.respond_to?(:mapped_label_name) }.map { |a| a.mapped_label_name.to_sym }
     end



.. _`Neo4j/ActiveNode/Labels/ClassMethods#size`:

**#size**
  

  .. code-block:: ruby

     def count(distinct = nil)
       fail(InvalidParameterError, ':count accepts `distinct` or nil as a parameter') unless distinct.nil? || distinct == :distinct
       q = distinct.nil? ? 'n' : 'DISTINCT n'
       self.query_as(:n).return("count(#{q}) AS count").first.count
     end





