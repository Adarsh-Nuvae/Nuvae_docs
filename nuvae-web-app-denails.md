<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Next.js to Django Mapping Guide</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }

        .header {
            text-align: center;
            color: white;
            margin-bottom: 30px;
            padding: 20px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
        }

        .header h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }

        .overview-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .overview-card {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
            border: 1px solid rgba(255, 255, 255, 0.18);
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }

        .overview-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
        }

        .overview-card h3 {
            color: #4a5568;
            margin-bottom: 15px;
            font-size: 1.3em;
            border-bottom: 2px solid #667eea;
            padding-bottom: 8px;
        }

        .mapping-item {
            display: flex;
            align-items: center;
            margin-bottom: 10px;
            padding: 8px;
            border-radius: 8px;
            background: rgba(102, 126, 234, 0.1);
        }

        .nextjs-code {
            background: #2d3748;
            color: #61dafb;
            padding: 4px 8px;
            border-radius: 4px;
            font-family: 'Courier New', monospace;
            margin-right: 10px;
            flex: 1;
        }

        .django-code {
            background: #0f4c3a;
            color: #10b981;
            padding: 4px 8px;
            border-radius: 4px;
            font-family: 'Courier New', monospace;
            flex: 1;
        }

        .arrow {
            margin: 0 10px;
            color: #667eea;
            font-weight: bold;
            font-size: 1.2em;
        }

        .section {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 15px;
            padding: 25px;
            margin-bottom: 25px;
            box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
            border: 1px solid rgba(255, 255, 255, 0.18);
        }

        .section-title {
            color: #2d3748;
            font-size: 1.8em;
            margin-bottom: 20px;
            padding: 15px;
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            border-radius: 10px;
            text-align: center;
            box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
        }

        .code-block {
            background: #1a202c;
            color: #e2e8f0;
            padding: 20px;
            border-radius: 10px;
            margin: 15px 0;
            overflow-x: auto;
            border-left: 4px solid #667eea;
            font-family: 'Courier New', monospace;
            line-height: 1.4;
        }

        .nextjs-block {
            border-left-color: #61dafb;
            background: linear-gradient(135deg, #1a202c, #2d3748);
        }

        .django-block {
            border-left-color: #10b981;
            background: linear-gradient(135deg, #064e3b, #065f46);
        }

        .highlight {
            background: rgba(255, 235, 59, 0.3);
            padding: 2px 4px;
            border-radius: 3px;
            font-weight: bold;
        }

        .file-header {
            background: linear-gradient(135deg, #4f46e5, #7c3aed);
            color: white;
            padding: 12px 20px;
            border-radius: 8px 8px 0 0;
            font-family: 'Courier New', monospace;
            font-weight: bold;
            margin-bottom: 0;
        }

        .line-explanation {
            background: #f7fafc;
            border-left: 4px solid #e2e8f0;
            padding: 15px;
            margin: 10px 0;
            border-radius: 0 8px 8px 0;
        }

        .line-number {
            color: #667eea;
            font-weight: bold;
            margin-right: 10px;
        }

        .concept-tag {
            display: inline-block;
            background: #667eea;
            color: white;
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 0.8em;
            margin: 2px;
        }

        .api-method {
            display: inline-block;
            padding: 4px 8px;
            border-radius: 4px;
            color: white;
            font-weight: bold;
            font-size: 0.9em;
            margin-right: 10px;
        }

        .get { background: #10b981; }
        .post { background: #f59e0b; }
        .put { background: #3b82f6; }
        .delete { background: #ef4444; }

        .toc {
            background: rgba(255, 255, 255, 0.9);
            border-radius: 15px;
            padding: 20px;
            margin-bottom: 30px;
            box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
        }

        .toc h2 {
            color: #4a5568;
            margin-bottom: 15px;
            text-align: center;
        }

        .toc ul {
            list-style: none;
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 10px;
        }

        .toc li {
            background: linear-gradient(135deg, #667eea, #764ba2);
            border-radius: 8px;
            transition: transform 0.2s ease;
        }

        .toc li:hover {
            transform: scale(1.05);
        }

        .toc a {
            color: white;
            text-decoration: none;
            padding: 12px 16px;
            display: block;
            border-radius: 8px;
        }

        @media (max-width: 768px) {
            .container {
                padding: 10px;
            }
            
            .overview-grid {
                grid-template-columns: 1fr;
            }
            
            .mapping-item {
                flex-direction: column;
                gap: 10px;
            }
            
            .nextjs-code, .django-code {
                margin: 0;
            }
            
            .arrow {
                transform: rotate(90deg);
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🚀 Next.js to Django Migration Guide</h1>
            <p>Complete mapping of Nuvae's denials API from Next.js to Django patterns</p>
        </div>

        <div class="toc">
            <h2>📚 Table of Contents</h2>
            <ul>
                <li><a href="#overview">Core Concept Mapping</a></li>
                <li><a href="#api-denials">GET /api/denials</a></li>
                <li><a href="#api-review">PUT /api/denials/[id]/review</a></li>
                <li><a href="#api-bulk">Bulk Review API</a></li>
                <li><a href="#api-reasons">Denial Reasons CRUD</a></li>
                <li><a href="#page-details">Denial Details Page</a></li>
                <li><a href="#client-helpers">Frontend Client Helpers</a></li>
                <li><a href="#django-equivalent">Complete Django Implementation</a></li>
            </ul>
        </div>

        <div id="overview" class="section">
            <h2 class="section-title">🎯 Core Concept Mapping</h2>
            <div class="overview-grid">
                <div class="overview-card">
                    <h3>🛣️ Routing & Views</h3>
                    <div class="mapping-item">
                        <div class="nextjs-code">src/pages/api/denials/index.ts</div>
                        <div class="arrow">→</div>
                        <div class="django-code">urls.py + views.py</div>
                    </div>
                    <div class="mapping-item">
                        <div class="nextjs-code">withAPIAuth({GET, PUT})</div>
                        <div class="arrow">→</div>
                        <div class="django-code">@permission_required</div>
                    </div>
                </div>

                <div class="overview-card">
                    <h3>🗄️ Database & ORM</h3>
                    <div class="mapping-item">
                        <div class="nextjs-code">prisma.denial.findMany()</div>
                        <div class="arrow">→</div>
                        <div class="django-code">Denial.objects.filter()</div>
                    </div>
                    <div class="mapping-item">
                        <div class="nextjs-code">include: { patient: true }</div>
                        <div class="arrow">→</div>
                        <div class="django-code">select_related('patient')</div>
                    </div>
                </div>

                <div class="overview-card">
                    <h3>🔐 Authentication</h3>
                    <div class="mapping-item">
                        <div class="nextjs-code">requiredPermissions: ["view_denials"]</div>
                        <div class="arrow">→</div>
                        <div class="django-code">@permission_required('app.view_denial')</div>
                    </div>
                    <div class="mapping-item">
                        <div class="nextjs-code">withNavigationAuth()</div>
                        <div class="arrow">→</div>
                        <div class="django-code">@login_required</div>
                    </div>
                </div>

                <div class="overview-card">
                    <h3>📄 Page Rendering</h3>
                    <div class="mapping-item">
                        <div class="nextjs-code">getServerSideProps</div>
                        <div class="arrow">→</div>
                        <div class="django-code">view context building</div>
                    </div>
                    <div class="mapping-item">
                        <div class="nextjs-code">React Component</div>
                        <div class="arrow">→</div>
                        <div class="django-code">Django Template</div>
                    </div>
                </div>
            </div>
        </div>

        <div id="api-denials" class="section">
            <h2 class="section-title">📋 API: GET /api/denials</h2>
            <div class="file-header">src/pages/api/denials/index.ts</div>
            
            <div class="line-explanation">
                <span class="line-number">Lines 3-13:</span> 
                <span class="concept-tag">Imports</span>
                <div class="code-block nextjs-block">
// Next.js imports
import { NextApiRequest, NextApiResponse } from 'next'
import { Denial, DenialStatus, Prisma } from '@prisma/client'
import prisma from '/lib/db'
import { withAPIAuth } from '/lib/auth'
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django imports
from django.http import JsonResponse
from django.contrib.auth.decorators import permission_required
from .models import Denial, DenialStatus
from django.db.models import Q, F
                </div>
            </div>

            <div class="line-explanation">
                <span class="line-number">Lines 19-54:</span>
                <span class="concept-tag">Query Params Parsing</span>
                <div class="code-block nextjs-block">
// Next.js: Extract query parameters
const {
  pageNo = 1,
  pageSize = 20,
  searchQuery,
  payer = [],
  status,
  startDate,
  endDate,
  appealAmount,
  financialClass = []
} = req.query as GetDenialsParams
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Parse request parameters
page_no = int(request.GET.get('pageNo', 1))
page_size = int(request.GET.get('pageSize', 20))
search_query = request.GET.get('searchQuery')
payer = request.GET.getlist('payer[]')
status = request.GET.get('status')
start_date = request.GET.get('startDate')
end_date = request.GET.get('endDate')
financial_class = request.GET.getlist('financialClass[]')
                </div>
            </div>

            <div class="line-explanation">
                <span class="line-number">Lines 57-64:</span>
                <span class="concept-tag">Base Query Filters</span>
                <div class="code-block nextjs-block">
// Next.js: Build Prisma where clause
const whereClause: Prisma.DenialWhereInput = {
  isAppealable: true,
  ...(payer?.length > 0 && {
    denialPlan: { denialPayerId: { in: payer.map(Number) } }
  })
}
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Build query filters
filters = {'is_appealable': True}
if payer:
    filters['denial_plan__denial_payer_id__in'] = [int(p) for p in payer]

queryset = Denial.objects.filter(**filters)
                </div>
            </div>

            <div class="line-explanation">
                <span class="line-number">Lines 65-69:</span>
                <span class="concept-tag">Search Filter</span>
                <div class="code-block nextjs-block">
// Next.js: Add search filter
if (searchQuery) {
  whereClause.patientAccountNumber = {
    contains: searchQuery,
    mode: 'insensitive'
  }
}
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Add search filter
if search_query:
    queryset = queryset.filter(
        patient_account_number__icontains=search_query
    )
                </div>
            </div>

            <div class="line-explanation">
                <span class="line-number">Lines 121-137:</span>
                <span class="concept-tag">Database Query Execution</span>
                <div class="code-block nextjs-block">
// Next.js: Execute count and paged query
const [actualTotalCount, denialsResult] = await Promise.all([
  totalCount ?? prisma.denial.count({ where: whereClause }),
  prisma.denial.findMany({
    where: whereClause,
    skip: (pageNo - 1) * pageSize,
    take: pageSize,
    orderBy: { createdAt: 'desc' },
    include: {
      patient: { select: { resource: true } },
      financialClass: true
    }
  })
])
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Execute query with pagination
total_count = queryset.count()
offset = (page_no - 1) * page_size

denials = queryset.select_related(
    'financial_class'
).prefetch_related(
    'patient'
).order_by('-created_at')[offset:offset + page_size]
                </div>
            </div>

            <div class="line-explanation">
                <span class="line-number">Lines 145-150:</span>
                <span class="concept-tag">Response Formation</span>
                <div class="code-block nextjs-block">
// Next.js: Return JSON response
const response = {
  denials: denialDtos,
  totalClaimsCount: actualTotalCount,
  pageNo,
  pageSize
}
return res.json(response)
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Return JSON response
response_data = {
    'denials': [denial_to_dto(d) for d in denials],
    'totalClaimsCount': total_count,
    'pageNo': page_no,
    'pageSize': page_size
}
return JsonResponse(response_data, safe=False)
                </div>
            </div>
        </div>

        <div id="api-review" class="section">
            <h2 class="section-title">✅ API: PUT /api/denials/[denialId]/review</h2>
            <div class="file-header">src/pages/api/denials/[denialId]/review/index.ts</div>

            <div class="line-explanation">
                <span class="line-number">Lines 8-9:</span>
                <span class="concept-tag">Route Parameters</span>
                <div class="code-block nextjs-block">
// Next.js: Extract route params and body
const { denialId } = req.query
const { files = [] } = req.body
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Extract URL parameters and request data
def review_denial(request, denial_id):
    import json
    data = json.loads(request.body)
    files = data.get('files', [])
                </div>
            </div>

            <div class="line-explanation">
                <span class="line-number">Lines 13-19:</span>
                <span class="concept-tag">Record Lookup</span>
                <div class="code-block nextjs-block">
// Next.js: Find denial by ID
const denial = await prisma.denial.findUnique({
  where: { id: Number(denialId) }
})

if (!denial) {
  return res.status(404).json({ error: 'Denial not found' })
}
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Find denial or return 404
from django.shortcuts import get_object_or_404

denial = get_object_or_404(Denial, id=int(denial_id))
                </div>
            </div>

            <div class="line-explanation">
                <span class="line-number">Lines 27-33:</span>
                <span class="concept-tag">Status Update</span>
                <div class="code-block nextjs-block">
// Next.js: Update denial status
const updatedDenial = await prisma.denial.update({
  where: { id: Number(denialId) },
  data: {
    status: DenialStatus.CLOSED,
    updatedAt: new Date()
  }
})
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Update denial status
from django.utils import timezone

Denial.objects.filter(id=int(denial_id)).update(
    status=DenialStatus.CLOSED,
    updated_at=timezone.now()
)
                </div>
            </div>

            <div class="line-explanation">
                <span class="line-number">Lines 35-49:</span>
                <span class="concept-tag">Related Record Updates</span>
                <div class="code-block nextjs-block">
// Next.js: Update related PDR forms and appeals
if (files.includes(DenialFileType.PDR_FORM)) {
  await prisma.pDRForm.updateMany({
    where: { denialId: Number(denialId) },
    data: { denialStatus: DenialStatus.CLOSED }
  })
}

if (files.includes(DenialFileType.APPEAL_LETTER)) {
  await prisma.appeal.updateMany({
    where: { denialId: Number(denialId) },
    data: { denialStatus: DenialStatus.CLOSED }
  })
}
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Update related records
if 'PDR_FORM' in files:
    PDRForm.objects.filter(denial_id=int(denial_id)).update(
        denial_status=DenialStatus.CLOSED
    )

if 'APPEAL_LETTER' in files:
    Appeal.objects.filter(denial_id=int(denial_id)).update(
        denial_status=DenialStatus.CLOSED
    )
                </div>
            </div>
        </div>

        <div id="api-bulk" class="section">
            <h2 class="section-title">📦 API: Bulk Review Operations</h2>
            <div class="file-header">src/pages/api/denials/bulk-review/index.ts</div>

            <div class="line-explanation">
                <span class="concept-tag">GET Method</span>
                <span class="concept-tag">Bulk Statistics</span>
                <div class="code-block nextjs-block">
// Next.js: Calculate bulk review statistics
const [totalCount, totalReviewedCount, totalEditedCount] = await Promise.all([
  prisma.denial.count({ where: whereClause }),
  prisma.denial.count({ 
    where: { ...whereClause, status: DenialStatus.CLOSED } 
  }),
  prisma.denial.count({ 
    where: { ...whereClause, status: DenialStatus.INPROGRESS } 
  })
])

const totalPendingReview = totalCount - totalReviewedCount - totalEditedCount
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Calculate bulk statistics with aggregation
from django.db.models import Count, Case, When, IntegerField

stats = Denial.objects.filter(**base_filters).aggregate(
    total_count=Count('id'),
    reviewed_count=Count(
        Case(When(status=DenialStatus.CLOSED, then=1))
    ),
    edited_count=Count(
        Case(When(status=DenialStatus.INPROGRESS, then=1))
    )
)

pending_count = stats['total_count'] - stats['reviewed_count'] - stats['edited_count']
                </div>
            </div>

            <div class="line-explanation">
                <span class="concept-tag">PUT Method</span>
                <span class="concept-tag">Bulk Update</span>
                <div class="code-block nextjs-block">
// Next.js: Bulk update denials
const updateResult = await prisma.denial.updateMany({
  where: {
    ...whereClause,
    status: { not: DenialStatus.RESOLVED }
  },
  data: {
    status: DenialStatus.CLOSED,
    updatedAt: new Date()
  }
})
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Bulk update with exclusion
updated_count = Denial.objects.filter(
    **filters
).exclude(
    status=DenialStatus.RESOLVED
).update(
    status=DenialStatus.CLOSED,
    updated_at=timezone.now()
)
                </div>
            </div>
        </div>

        <div id="api-reasons" class="section">
            <h2 class="section-title">📝 API: Denial Reasons CRUD</h2>
            <div class="file-header">src/pages/api/denials/[denialId]/reasons.ts</div>

            <div class="line-explanation">
                <span class="concept-tag">CRUD Operations</span>
                <div class="code-block nextjs-block">
// Next.js: Method-based routing in single file
export default async function handler(req, res) {
  switch (req.method) {
    case 'GET':
      return req.query.id 
        ? await getClaimRejectionReason(req, res)
        : await getClaimRejectionReasons(req, res)
    case 'POST':
      return await createClaimRejectionReason(req, res)
    case 'PUT':
      return await updateClaimRejectionReason(req, res)
    case 'DELETE':
      return await deleteClaimRejectionReason(req, res)
    default:
      return res.status(405).json({ error: 'Method not allowed' })
  }
}
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: URLs and ViewSet approach
# urls.py
from rest_framework.routers import DefaultRouter
from .views import ClaimRejectionReasonViewSet

router = DefaultRouter()
router.register(
    r'denials/(?P<denial_id>[^/.]+)/reasons',
    ClaimRejectionReasonViewSet,
    basename='denial-reasons'
)

# views.py
class ClaimRejectionReasonViewSet(viewsets.ModelViewSet):
    def get_queryset(self):
        return ClaimRejectionReason.objects.filter(
            denial_id=self.kwargs['denial_id']
        ).select_related('denial').prefetch_related(
            'relevant_documents__knowledge_reference__document',
            'icd_codes'
        )
                </div>
            </div>

            <div class="line-explanation">
                <span class="concept-tag">Create Operation</span>
                <div class="code-block nextjs-block">
// Next.js: Create new rejection reason
const result = await prisma.claimRejectionReason.create({
  data: {
    denialId: Number(denialId),
    text: content,
    accuracy: accuracy ?? 0
  }
})
return res.status(201).json(result)
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: Create with validation
def create(self, request, *args, **kwargs):
    serializer = self.get_serializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    
    reason = ClaimRejectionReason.objects.create(
        denial_id=int(kwargs['denial_id']),
        text=serializer.validated_data['content'],
        accuracy=serializer.validated_data.get('accuracy', 0)
    )
    
    return Response(
        ClaimRejectionReasonSerializer(reason).data,
        status=status.HTTP_201_CREATED
    )
                </div>
            </div>
        </div>

        <div id="page-details" class="section">
            <h2 class="section-title">📄 Page: Denial Details</h2>
            <div class="file-header">src/pages/denials/[denialId]/index.tsx</div>

            <div class="line-explanation">
                <span class="concept-tag">Server-Side Data Fetching</span>
                <div class="code-block nextjs-block">
// Next.js: getServerSideProps for SSR
export const getServerSideProps = async (context) => {
  const { denialId } = context.params
  
  const denial = await prisma.denial.findUnique({
    where: { id: Number(denialId) },
    include: {
      patient: { select: { resource: true } },
      financialClass: true,
      denialPlan: { include: { denialPayer: true } }
    }
  })

  const denialDto = DenialDtoAdapter(
    DenialClaimAdapter,
    DenialPatientAdapter,
    DenialServiceAdapter
  ).toDenialDto(denial).toJson()

  return { props: { denial: denialDto } }
}
                </div>
                <p><strong>Django Equivalent:</strong></p>
                <div class="code-block django-block">
# Django: View with context building
def denial_detail_view(request, denial_id):
    denial = get_object_or_404(
        Denial.objects.select_related(
            'financial_class',
            'denial_plan__denial_payer',
            'patient'
        ),
        id=denial_id
    )
    
    # Convert to DTO/context data
    denial_dto = Denial
