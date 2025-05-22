# ai-powered-youtube-comment-reorganization
ai powered comment reader
const SmartYouTubeComments = () => {
  const [currentTime, setCurrentTime] = useState(0);
  const [isPlaying, setIsPlaying] = useState(false);
  const [activeFilter, setActiveFilter] = useState('all');
  const [selectedTimestamp, setSelectedTimestamp] = useState(null);
  const [showFloatingComments, setShowFloatingComments] = useState(true);
  const [expandedThreads, setExpandedThreads] = useState(new Set());
  const videoRef = useRef(null);

  // Mock video duration (5 minutes)
  const videoDuration = 300;

  // Mock comments data with AI-generated clusters
  const comments = [
    {
      id: 1,
      author: 'TechReviewer99',
      text: 'The explanation at 0:45 about the algorithm is brilliant!',
      timestamp: 45,
      likes: 127,
      replies: 8,
      topic: 'algorithm',
      sentiment: 'positive',
      cluster: 'technical-discussion'
    },
    {
      id: 2,
      author: 'CodeMaster',
      text: 'Wait, I think there\'s an error in the code shown at 1:20',
      timestamp: 80,
      likes: 43,
      replies: 12,
      topic: 'code-review',
      sentiment: 'critical',
      cluster: 'technical-discussion'
    },
    {
      id: 3,
      author: 'BeginnerDev',
      text: 'Could someone explain the part at 2:15? I\'m confused about the implementation',
      timestamp: 135,
      likes: 89,
      replies: 15,
      topic: 'help-request',
      sentiment: 'neutral',
      cluster: 'learning-support'
    },
    {
      id: 4,
      author: 'AIEnthusiast',
      text: 'The AI integration shown at 3:30 is mind-blowing! This changes everything.',
      timestamp: 210,
      likes: 234,
      replies: 22,
      topic: 'ai-discussion',
      sentiment: 'positive',
      cluster: 'innovation-talk'
    },
    {
      id: 5,
      author: 'SkepticalDev',
      text: 'I disagree with the approach at 3:45. There are better solutions.',
      timestamp: 225,
      likes: 67,
      replies: 18,
      topic: 'alternative-solutions',
      sentiment: 'critical',
      cluster: 'technical-discussion'
    },
    {
      id: 6,
      author: 'MobileFirst',
      text: 'Great tutorial! The mobile optimization tips at 4:20 are exactly what I needed.',
      timestamp: 260,
      likes: 156,
      replies: 6,
      topic: 'mobile-dev',
      sentiment: 'positive',
      cluster: 'practical-tips'
    }
  ];

  // AI-generated topic clusters
  const topicClusters = {
    'technical-discussion': { label: 'Technical Discussion', color: 'bg-blue-100 text-blue-800', count: 3 },
    'learning-support': { label: 'Learning & Help', color: 'bg-green-100 text-green-800', count: 1 },
    'innovation-talk': { label: 'Innovation Talk', color: 'bg-purple-100 text-purple-800', count: 1 },
    'practical-tips': { label: 'Practical Tips', color: 'bg-orange-100 text-orange-800', count: 1 }
  };

  // Get comments relevant to current video time (±30 seconds)
  const getContextualComments = () => {
    const timeWindow = 30;
    return comments.filter(comment => 
      Math.abs(comment.timestamp - currentTime) <= timeWindow
    ).sort((a, b) => Math.abs(a.timestamp - currentTime) - Math.abs(b.timestamp - currentTime));
  };

  // Get comments filtered by active filter
  const getFilteredComments = () => {
    let filtered = comments;
    
    if (activeFilter !== 'all') {
      filtered = comments.filter(comment => comment.cluster === activeFilter);
    }
    
    return filtered.sort((a, b) => b.likes - a.likes);
  };

  // Format timestamp for display
  const formatTime = (seconds) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins}:${secs.toString().padStart(2, '0')}`;
  };

  // Simulate video playback
  useEffect(() => {
    let interval;
    if (isPlaying) {
      interval = setInterval(() => {
        setCurrentTime(prev => (prev + 1) % videoDuration);
      }, 1000);
    }
    return () => clearInterval(interval);
  }, [isPlaying, videoDuration]);

  const togglePlay = () => {
    setIsPlaying(!isPlaying);
  };

  const seekToTime = (timestamp) => {
    setCurrentTime(timestamp);
    setSelectedTimestamp(timestamp);
  };

  const toggleThread = (commentId) => {
    const newExpanded = new Set(expandedThreads);
    if (newExpanded.has(commentId)) {
      newExpanded.delete(commentId);
    } else {
      newExpanded.add(commentId);
    }
    setExpandedThreads(newExpanded);
  };

  const FloatingComment = ({ comment }) => (
    <div className="bg-black bg-opacity-80 text-white text-sm p-2 rounded-lg mb-2 max-w-xs animate-pulse">
      <div className="flex items-center gap-2 mb-1">
        <span className="font-medium">{comment.author}</span>
        <span className="text-xs opacity-75">@{formatTime(comment.timestamp)}</span>
      </div>
      <p className="text-xs leading-tight">{comment.text}</p>
      <div className="flex items-center gap-2 mt-1">
        <ThumbsUp className="w-3 h-3" />
        <span className="text-xs">{comment.likes}</span>
      </div>
    </div>
  );

  const CommentCard = ({ comment, isContextual = false }) => (
    <div className={`border rounded-lg p-4 mb-3 transition-all duration-200 ${
      isContextual ? 'border-blue-300 bg-blue-50' : 'border-gray-200 bg-white'
    } ${selectedTimestamp === comment.timestamp ? 'ring-2 ring-blue-400' : ''}`}>
      <div className="flex items-start gap-3">
        <div className="w-8 h-8 bg-gray-300 rounded-full flex items-center justify-center text-sm font-medium">
          {comment.author[0]}
        </div>
        <div className="flex-1">
          <div className="flex items-center gap-2 mb-1">
            <span className="font-medium text-sm">{comment.author}</span>
            <button
              onClick={() => seekToTime(comment.timestamp)}
              className="flex items-center gap-1 text-xs text-blue-600 hover:text-blue-800 bg-blue-100 px-2 py-1 rounded-full"
            >
              <Clock className="w-3 h-3" />
              {formatTime(comment.timestamp)}
            </button>
            <span className={`text-xs px-2 py-1 rounded-full ${topicClusters[comment.cluster]?.color}`}>
              {topicClusters[comment.cluster]?.label}
            </span>
          </div>
          <p className="text-sm text-gray-700 mb-2">{comment.text}</p>
          <div className="flex items-center gap-4 text-xs text-gray-500">
            <button className="flex items-center gap-1 hover:text-blue-600">
              <ThumbsUp className="w-3 h-3" />
              {comment.likes}
            </button>
            <button 
              className="flex items-center gap-1 hover:text-blue-600"
              onClick={() => toggleThread(comment.id)}
            >
              <Reply className="w-3 h-3" />
              {comment.replies} replies
              {expandedThreads.has(comment.id) ? 
                <ChevronUp className="w-3 h-3" /> : 
                <ChevronDown className="w-3 h-3" />
              }
            </button>
          </div>
          {expandedThreads.has(comment.id) && (
            <div className="mt-3 pl-4 border-l-2 border-gray-200">
              <div className="text-xs text-gray-500 mb-2">
                {comment.replies} replies (showing preview)
              </div>
              <div className="bg-gray-50 p-2 rounded text-sm">
                <span className="font-medium">DevHelper:</span> Great question! Here's how you can implement it...
              </div>
            </div>
          )}
        </div>
      </div>
    </div>
  );

  const contextualComments = getContextualComments();
  const filteredComments = getFilteredComments();

  return (
    <div className="max-w-4xl mx-auto bg-gray-50 min-h-screen">
      {/* Mock Video Player */}
      <div className="relative bg-black aspect-video">
        <div className="absolute inset-0 flex items-center justify-center text-white text-xl">
          Mock Video Player
        </div>
        
        {/* Video Controls */}
        <div className="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black to-transparent p-4">
          <div className="flex items-center gap-4">
            <button
              onClick={togglePlay}
              className="text-white hover:text-blue-400 transition-colors"
            >
              {isPlaying ? <Pause className="w-6 h-6" /> : <Play className="w-6 h-6" />}
            </button>
            <div className="flex-1">
              <div className="bg-gray-600 h-1 rounded-full">
                <div 
                  className="bg-blue-500 h-1 rounded-full transition-all duration-1000"
                  style={{ width: `${(currentTime / videoDuration) * 100}%` }}
                />
              </div>
            </div>
            <span className="text-white text-sm">
              {formatTime(currentTime)} / {formatTime(videoDuration)}
            </span>
          </div>
        </div>

        {/* Floating Comments Overlay */}
        {showFloatingComments && contextualComments.length > 0 && (
          <div className="absolute top-4 right-4 max-w-xs">
            <div className="mb-2">
              <button
                onClick={() => setShowFloatingComments(false)}
                className="text-white text-xs bg-black bg-opacity-50 px-2 py-1 rounded"
              >
                Hide Live Comments
              </button>
            </div>
            {contextualComments.slice(0, 2).map(comment => (
              <FloatingComment key={comment.id} comment={comment} />
            ))}
          </div>
        )}
      </div>

      {/* Comment Section */}
      <div className="p-4">
        {/* Header */}
        <div className="flex items-center justify-between mb-4">
          <h2 className="text-xl font-semibold flex items-center gap-2">
            <MessageCircle className="w-5 h-5" />
            Smart Comments
          </h2>
          {!showFloatingComments && (
            <button
              onClick={() => setShowFloatingComments(true)}
              className="text-sm text-blue-600 hover:text-blue-800"
            >
              Show Live Comments
            </button>
          )}
        </div>

        {/* Topic Filter Pills */}
        <div className="flex gap-2 mb-4 overflow-x-auto pb-2">
          <button
            onClick={() => setActiveFilter('all')}
            className={`px-3 py-1 rounded-full text-sm font-medium whitespace-nowrap transition-colors ${
              activeFilter === 'all' 
                ? 'bg-blue-600 text-white' 
                : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
            }`}
          >
            All Comments ({comments.length})
          </button>
          {Object.entries(topicClusters).map(([key, cluster]) => (
            <button
              key={key}
              onClick={() => setActiveFilter(key)}
              className={`px-3 py-1 rounded-full text-sm font-medium whitespace-nowrap transition-colors ${
                activeFilter === key 
                  ? 'bg-blue-600 text-white' 
                  : `${cluster.color} hover:opacity-80`
              }`}
            >
              {cluster.label} ({cluster.count})
            </button>
          ))}
        </div>

        {/* Contextual Comments Section */}
        {contextualComments.length > 0 && (
          <div className="mb-6">
            <h3 className="text-lg font-medium mb-3 flex items-center gap-2">
              <Clock className="w-4 h-4 text-blue-600" />
              Comments for Current Time ({formatTime(currentTime)})
            </h3>
            {contextualComments.map(comment => (
              <CommentCard key={comment.id} comment={comment} isContextual={true} />
            ))}
          </div>
        )}

        {/* All Comments Section */}
        <div>
          <h3 className="text-lg font-medium mb-3 flex items-center gap-2">
            <Users className="w-4 h-4 text-gray-600" />
            {activeFilter === 'all' ? 'All Comments' : `${topicClusters[activeFilter]?.label} Comments`}
          </h3>
          {filteredComments.map(comment => (
            <CommentCard key={comment.id} comment={comment} />
          ))}
        </div>
      </div>
    </div>
  );
};

export default SmartYouTubeComments;
